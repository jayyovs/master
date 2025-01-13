# DataGrip Tips, Shortcuts, & Setup Guidance

## **General Tips**
- lowercase a word (either highlighted or have your cursor touching the word) -> ctrl + shift + U
- If copying code from the dbt project to DataGrip and want to run it, you'll need to replace all 'ref' calls with x_schema:
	- In DataGrip, press ctrl + R
	- Make sure 'regex' is marked off (the `.*` to the right of the pane is highlighted)
	- Enter this in the top section: `(\{\{ ref\(')(\w+)('\) \}\})`
	- Enter this in the bottom section: `x_schema\.$2`
	- Click 'replace all'
- If you are taking code from DataGrip and want to make it usable by dbt with the 'ref' calls added back in:
	- In DataGrip, press ctrl + R
	- Make sure 'regex' is marked off (the `.*` to the right of the pane is highlighted)
	- Enter this in the top section: `(x_schema\.)(\w+)`
	- Enter this in the bottom section: `{{ ref('$2') }}`
	- Click 'replace all'

## **IDE Settings**

- Navigate to **File > Settings > Editor > Code Style > SQL > General:**
	- Under **Case**:
		- Keywords: To lower
		- Identifiers: To lower
		- Built-in types: To lower
		- Custom types: To lower
		- Aliases: Do not change
		- Built-in functions: To lower
		- Quoted identifiers: Do not change
		- Use original case: Unchecked
		
	- Under **Queries**:
		- Align the first word of clause: To left
		- Place clause elements on: Same line
		- Place comma: Auto
		- Collapse short statement: Subqueries only
		- Keep section elements under section header: check off
		- Align section elements: check off
		- Align line comments at right of elements: check off
		
	- Under **Tabs and Indents**:
		- Tab size: 2
		- Indent: 2
		- Continuation indent: 2
		- Keep indents on empty lines: Unchecked
