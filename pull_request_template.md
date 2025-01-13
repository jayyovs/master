## (Mandatory) Was the model run in dbt? (yes/no)
_Confirm that `dbt run -s [model_name]+1` and `dbt test -s [model_name]+1` were successful_


## Description
<!---
Describe your changes and why you are making them. Also link to a Jira story or a related pull request, if applicable.
Quick Tip : If you prefix your branch name after the JIRA ticket id, the branch and commits will be automatically linked and visble on the ticket. For eg, AE-XX_update_marts.
-->


## Model validation
<!---
Describe what you have done to ensure your changes behave as expected.
You may also include screenshots of model output or a query that compares an existing model to a new/updated model in this change.
-->


## Documentation (optional)
<!---
In engineering, design ideas are always based on someone's earlier work, so documenting is important to put your work in context. If applicable, please create/add documentation in Confluence, detailing the work you did and assumptions that you might have taken.
-->


## Next steps (optional)
<!---
Does a dashboard or a data source need to be updated? Does someone in the company need to be notified? Note those here.
-->


## Checklist
<!---
Put an `x` in all the items that apply, make notes next to any that haven't been addressed, and remove any items that are not relevant to this PR.
-->
- [ ] My code follows our style guide
- [ ] I have added and run tests to new models and their dependencies.
- [ ] I added definitions in the yaml file to every column in every model that I added to the data warehouse
- [ ] I have added in-line comments to anything that is not overtly clear and updated existing comments to reflect relevant changes in code.
