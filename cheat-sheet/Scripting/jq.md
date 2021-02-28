Example-wise the jq manpage is not really helpful. Let's document some simple examples here...

To test queries live use [jqplay.org](https://jqplay.org/) or simply a local terminal window.

## Premise

Create `my.json` to use in the examples below:

    echo '{"timestamp":1234567890,"report":"AgeReport","results":[{"name":"John Doe","age":43,"city":"TownA"},{"name":"Joe Smith","age":10,"city":"TownB"}]}'>my.json

## Output Formatting

If you do only care about output formatting (pretty print) run:

    jq . my.json

or when using a pipeline:

    cat my.json | jq

Note - for redirection you need to pass a filter too to avoid a syntax error:

    jq . my.json > output.json

## jq Extraction Examples

Consider this example document

    {
        "timestamp": 1234567890,
        "report": "Age Report",
        "results": [
            { "name": "John Doe", "age": 43, "city": "TownA" },
            { "name": "Joe Smith",  "age": 10, "city": "TownB" }
        ]
    }

To extract top level attributes "timestamp" and "report":

    jq '. | {timestamp,report}' my.json

To extract name and age of each "results" item:

    jq '.results[] | {name, age}' my.json

Filter by attribute:

    # Get the age of 'John Doe':
    jq '.results[] | select(.name == "John Doe") | {age}' my.json
    # Get the complete records for all 'Joe Smith' aged 10:
    jq '.results[] | select((.name == "Joe Smith") and (.age = 10))' my.json
    # Get complete records for all names containing 'Jo':
    jq '.results[] | select(.name | contains("Jo"))' my.json
    # Same as previous but return as an array:
    jq '[.results[] | select(.name | contains("Jo"))]' my.json
    # Get complete records for all names matching PCRE regex 'Joe\+Smith':
    jq '.results[] | select(.name | test("Joe\\s+Smith"))' my.json
    
## "Deep" Value Extraction

If you want to combine subkeys at different levels it won't work like this

    jq '.items[] | { metadata["created"], name }'

Instead you can access values like this

    jq '.items[] | .metadata["created"], .name'

The drawback being, that you do not get a JSON output, but each value on a new line.

## Changing values with jq

Merging/overwriting keys

    echo '{ "a": 1, "b": 2 }' |\
    jq '. |= . + {
      "c": 3
    }'

Adding elements to lists

    echo '{ "names": ["Marie", "Sophie"] }' |\
    jq '.names |= .+ [
       "Natalie"
    ]'   

## Handle Empty Arrays

When you want to iterate and an array you access is empty you get something like

    jq: error (at <stdin>:3): Cannot iterate over null (null)

To workaround the optional array protect the access with

    select(.my_array | length > 0)
    
## Testing Types

    $ echo '[true, null, 42, "hello", []]' | ./jq 'map(type)'
    ["boolean","null","number","string","array"]
    
## Extracting key names

Given an JSON object like this

    {
       "animals": [
           "dog": { },
           "cat": { }
         ]
    }

you can extract the names of the animals using

    jq '.animals | keys'   

## Using jq in Shell Scripts

From https://www.terraform.io/docs/providers/external/data_source.html

### Parsing JSON into env vars

To fill environment variables from JSON object keys (e.g. $FOO from jq query ".foo")

    export $(jq -r '@sh "FOO=\(.foo) BAZ=\(.baz)"')

To make a bash array
 
    read -a bash_array < <(jq -r .|arrays|select(.!=null)|@tsv)
    
### JSON template using env vars

To create proper JSON from a shell script and properly escape variables:

    jq -n --arg foobaz "$FOOBAZ" '{"foobaz":$foobaz}'

### URL Encode
Quick easy way to url encode something
 
    date | jq -sRr @uri
