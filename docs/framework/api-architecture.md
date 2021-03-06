# API architecture


The term 'API' refers to that code stored in the top level api folder.
It's important to note that only compliant code & usage will be
supported. Usage documentation is in the [API
v3](/api/index.md) page.

## Standards {:#standards}

-   Interacting with code in the `api` folder *from outside the `api` folder* is only supported through the following means:
    * PHP: by using the `civicrm_api()` wrapper (or preferably the `civicrm_api3()` wrapper)
    * JavaScript: by using `CRM.api()` (or preferably `CRM.api3()`);
    * Smarty: by using the [Smarty interface](https://wiki.civicrm.org/confluence/display/CRMDOC/Smarty+API+interface)
    * REST: by using the [REST interface](https://wiki.civicrm.org/confluence/display/CRMDOC/REST+interface)
-   Functionality delivered by the API is only supported if it is
    1.  Advertised via the 'getfields function' OR (preferably AND)
    2.  Verified by a test
-   Where functions sit on the api but don't conform to the api
    standards they may be supported by the creators but not by the
    api team.

## Individual functions

-   There are a number of (fairly new) helper functions you should use
    in the test - please check [Writing a PHPUnit testcase
    HOWTO](/testing/setup.md)
-   API functions should match the relevant BAO names & have the
    functions 'get', 'create' and delete'
-   All functions should receive $params as an array(not a reference)
-   All functions should return either
    -   an id indexed array of results via
        the `civicrm_api3_create_success()` function
    -   an appropriate message via throw new api_Exception
-   (REVIEW this in light of feedback from bgm that it is not optimal
    for translation) Do not use `ts()` in error messages (this will be
    added in the exception function if you use api_Exception - but you
    need to add a code)
-   If a BAO object exists this must be passed to the `create_success` or
    `create_error` function for freeing

-   All setting of defaults, enforcement of required fields, management
    of field aliases & field validation should be done at the wrapper
    layer using the getfields function results
-   Dates will be formatted to ISO format at the api layer, individual
    functions should assume this (api layer uses `strtotime()`)
-   API functions should be accompanied by a `_spec` function to declare
    fields & specifications for fields not retrieved from the
    `$dao->fields` function or with different specifications to
    those returned.
-   All API functions should have a code comment block per standard
    below
-   As little functionality as possible should be in the API
    functions themselves. Single line api functions should be the rule
    not the exception. Where possible they should look like the
    following basic functions:

    ```php
    function civicrm_api3_survey_create($params) {
      return _civicrm_api3_basic_create(_civicrm_api3_get_BAO(__FUNCTION__), $params);
    }
    ```
    
    ```php
      function civicrm_api3_survey_get($params) {
        return _civicrm_api3_basic_get(_civicrm_api3_get_BAO(__FUNCTION__), $params);
      }
    ```
    
    ```php
    function civicrm_api3_survey_delete($params) {
      return _civicrm_api3_basic_delete(_civicrm_api3_get_BAO(__FUNCTION__), $params);
    }
    ```
    
## Code comment blocks {:#docblock}

-   API comment blocks shall include a description of the action & the
    input and output params - per the example
-   API comment blocks shall include references to test generated
    examples (these are generated by adding
    `$this->documentMe($params,$result,__FUNCTION__,__FILE__);`
    to tests

    ```php
    /**
     * Create or update a survey
     *
     * @param array $params 
     *   Associative array of property name/value pairs to insert in new 'survey'.
     * @example SurveyCreate.php Std Create example
     * @return array api result array
     * {@getfields survey_create}
     * @access public
     */
    ```
    
## `_spec` functions {:#spec}

A "`_spec` function" must exist for every API entity/action pair.

For example is is the [source](https://github.com/civicrm/civicrm-core/blob/1f4ea7262d865c72e8946481fdbfe18f5159da9e/api/v3/Tag.php#L62) of the `_spec` function for the `tag` entity, with the `create` action:

```php
function _civicrm_api3_tag_create_spec(&$params) {
  $params['used_for']['api.default'] = 'civicrm_contact';
  $params['name']['api.required'] = 1;
  $params['id']['api.aliases'] = array('tag');
}
```

If the `_spec` function makes no alterations to `$params`, then each field will behave as defined in the [schema definition](framework/schema-definition/#table-field) for the entity. All of those field-level xml schema tags are also available for use in the API `_spec` function. So for example, you can use `$params['id']['type'] = CRM_Utils_Type::T_INT;` to specify that the `'id'` field must be an integer.
  

Additionally, the following settings may be applied to each field: 

| Key | Description |
| -- | -- |
| `'api.aliases'` | An array of strings which represent alternate names by which this field may be referenced |
| `'api.default'` | Specifies a default value for the field |
| `'api.filter'` | ??? |
| `'api.required'` | Set this to `1` to indicate that the field is required |
| `'api.return'` | ??? |
| `'api.unique'` | ??? |
| `'description'` | A human-readable description of the field which will be displayed in the [API explorer](/api/index.md#api-explorer) |
| `'options'` | Used to specify an array of values which are considered valid for the field |
| `'supports_joins'` | ??? |
| `'FKApiName` | ??? |
| `'FKClassName'` | Allows the wrapper layer to return useful information if the constraint causes the call to fail. Should be a class name as a string. |
| `'FKKeyColumn'` | ??? |

## Tips for writing an API

-   **Use [civix generate:api](/extensions/civix.md#generate-api)** if writing a custom API within an extension.

-   **Look at existing apis** in the `civicrm/api/v3` directory
    as examples.  Start with a simple API like the survey api as the
    basis for your new api.
    
-   **Use [`_spec` functions](#spec)** to declare any fields or features

-   **Follow the naming patterns** for the entity, action, function signature, and any calls made to the API.

    - For example:
        -   PHP call: `civicrm_api('ExampleEntity', 'Create')`
        -   Ajax call: `/civicrm/ajax/rest?json=1&sequential=1&debug=1&entity=ExampleEntity&action=create`
        -   PHP file: `civicrm/api/v3/ExampleEntity.php` (or sometimes `civicrm/api/v3/ExampleEntity/Create.php`)
        -   Function signature:
            ```php
            function civicrm_api3_example_entity_create($params) {
            
            }
            ```

    -   **There is a simple relationship among the names of all of these
        elements** and if you name your php files and functions
        correctly and place them in the `civicrm/api/v3` directory, the
        API and function will automatically appear and work.  If you
        have created the files and functions but they do not appear in
        API explorer, it is most likely because of a mis-match in your
        naming conventions.
    -   **Note the change of case and use of underline characters** in
        filenames vs function names.  In filenames, an upper case letter
        denotes the start of a word, whereas in function names, all
        words are lower case and words are separated with underscores.
         To make matters more confusing, case of API names is important
        but case of the action name ('Action' in the example above)
        is (sometimes) ignored.

-   **Note that some actions (eg. getfields) are generic** and don't
    need to be re-defined.

-   **[Write a test](/testing/phpunit.md).**
-   **Write your [comment blocks](#docblock)**.
