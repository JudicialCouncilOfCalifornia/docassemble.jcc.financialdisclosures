# Porting CL interviews into our DA instance

1. Copy the YML from the CL instance. You'll probably want to put this into the Playground to ensure it loads correctly (try to run it).

2. Enable our CSS by adding the following to the top of the file:

        ---
        features:
            bootstrap theme: style.css

3. Just before the "Please click the link(s) below to download your filled out"..., add a code block to trigger the email send:

        ---
        mandatory: |
          True
        code: |
          if len(missing_files) > 0:
            send_email(to=party_email, template=incomplete_email)
          else:
            send_email(to=party_email, template=complete_email)

4. Add the email control block at the very bottom. NOTE: You need to change the `all_files` variable to match the filenames outputted by the script. In the section "Please click the link(s) below to download", you will find the correct variable names.

        ---
        template: incomplete_email
        subject: |
          FL-150: Documents Required
        content: |
          Thank you for the JCC's Financial Disclosure Toolkit to assist with your forms. We have used your responses to fill the FL-150 Income and Expenses Declaration.

          Right now your form is not complete, but we have provided a list of the remaining information you need.

          To download your incomplete form go here:

          ${fl150_with_attachments.url_for(temporary=True, seconds=60 * 60 * 24 * 10)}

          Here's the other information you may need to complete your form:

          % for proof in missing_files:
          ${proof}

          % endfor

          Once you complete the form, file it with your Court clerk.

          If you have any questions or feedback send an email to jack.madans-t@jud.ca.gov
        ---
        code: |
          all_files = [
            FL_150_73c36ecae5e3,
            Addendum_to_Income_and_Expense_Declaration_template_a60feaed58e5
          ]
          proofs = [
            "tax_return_proof",
            "past_employment_pay_stubs",
            "current_employer_pay_stubs1",
            "employer_pay_stubs2",
            "employer_pay_stubs3",
            "dividend_investment_proof",
            "rental_investment_proof",
            "trust_investment_proof",
            "other_investment_proof",
            "additional_income_proof",
            "business_income_proof",
            "business_income_proof2",
            "business_income_proof3",
            "extra_healthcare_proof",
            "major_loss_proof",
            "other_relationship_children_hardship_proof"
          ]
          missing_files = []
          all_vars = all_variables()
          for proof_name in proofs:
            proof_obj = all_vars.get(proof_name)
            if proof_obj is not None:
              if proof_obj != "None":
                all_files.append(value(proof_name))
              else:
                proof_title = proof_name.replace('_proof', '').replace('_', ' ').strip().title()
                missing_files.append(proof_title)

          fl150_with_attachments = pdf_concatenate(*all_files)
        ---
        template: complete_email
        subject: |
          FL-150: Completed Form
        content: |
          Thank you for the JCC's Financial Disclosure Toolkit to assist with your forms. We have used your responses to fill the FL-150 Income and Expenses Declaration.

          Right now your form is complete. To download your form go here:

          ${fl150_with_attachments.url_for(temporary=True, seconds=60 * 60 * 24 * 10)}

          Print your form and file it with your Court clerk, if you are ready.

          If you have any questions or feedback send an email to jack.madans-t@jud.ca.gov

