# Options

Extension to cover following requirement of EU24: "Information to be included in contract notices … 7. … Where appropriate, description of any options"

## Background

Directive 2014/24/EU requires Public contracts to set out clear ground rules. This in particular means: *"Information to be included in contract notices … 7. … Where appropriate, description of any options."*. Among other things, this requirement should also apply to clause **II.2.11 - Information about options**

At the moment there is already a draft of [ocds_options_extension](https://github.com/open-contracting-extensions/ocds_options_extension) by OCP,  designed to fulfill the above requirement, but this coverage is more formal. 

The new EU eForms have:

>- Options exist: yes/no
>- Description of options

And this is exactly what *ocds_options_extension* covers. But let's take a look at what we will have on a practical example: 

*II.2.7: Duration of the contract, framework agreement or dynamic purchasing system* of Contract Notice have to be filled as follows: *duration of contract in months* or *duration of contract in days* or *start: (dd/mm/yyyy) / End: (dd/mm/yyyy)* for contract. However, the Procuring Entity would like to provide himself with some room for maneuver by affording himself the right to extend the duration of the contract. This does not mean he is obliged to do so but he may consider that it may become a useful option to have (for further information, see Regulation 32(11) and 72(1)(a))

Designing this example with *ocds_options_extension* we will have the following:

```
{
  "tender": {
    "lots": [
      {
        "id": "1",
        "contractPeriod": {
          "durationInMonth":12
        },
        "options": "The buyer has the option to extend contract for three more month"
      }
    ]
  }
}
```

It's quite complicated to proceed on such information in an automated way but still, looks good ) 

Let's make an example a bit more complicated: let's assume that Procuring Entity wants to put an option for another one attribute that he obliged to indicate in the Contract Notice: *II.2.3) Place of performance*

```
{
  "tender": {
    "lots": [
      {
        "id": "1",
        "contractPeriod": {
          "durationInMonth":12
        },
        "placeOfPerformance":{
          "address":{},
          "NUTSCode":""
        },
        "options": "The buyer has the option to extend contract for three more month and by the way the place of performance can be changed  "
      }
    ]
  }
}
```

Now it is totally impossible to process this... So what to do? And this is where the idea of 'options' update rises 

## Approach

*Option* has to become a tool for Procuring Entity to put the options in a machine-readable way and be applicable for:

- tender.lot
- criterion.requirement
- target.observation

### Options for *Lot*

As shown in the example above the Procuring Entity may like to provide options for contract period and/or place of performance. Technically speaking it means that some attributes of *contractPeriod* and *palceOfPerformance* may have options. 

Lets design it using the following approach: 

1. *Lot* includes *optionDetails* with all the options details considered by PE for a number of attributes of *Lot* (in our case - two of them) presented by *optionGroups*
2. Each *optionGroup* has a relation to a specific attribute of *Lot* indicated by *relatesTo* attribute and set of available options for such related attribute
3. **Here basically the main trick**: since any attribute on which *optionGroup* may be related is either a property itself or an object with includes properties, *options* of such *optionsGroup* may vary the available value for such property. In other words - include the same properties together with values or being the property. 

```
{
  "tender": {
    "lots": [
      {
        "id": "1",
        "contractPeriod": {
          "durationInMonth": 12
        },
        "placeOfPerformance": {
          "address": {
            "country": "Ukraine"
          }
        },
        "optionDetails": {
          "optionGroups": [
            {
              "id": "1",
              "description": "",
              "relatesTo": "contractPeriod",
              "options": [
                {
                  "id": "1-1",
                  "durationInMonth": 15
                }
              ]
            },
            {
              "id": "2",
              "title": "",
              "description": "",
              "relatesTo": "placeOfPerformance",
              "options": [
                {
                  "id": "2-1",
                  "address": {
                    "country": "Moldova"
                  }
                },
                {
                  "id": "2-2",
                  "address": {
                    "country": "Romania"
                  }
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

To complete this framework - sort of *OptionsToCombine* array may be used to indicate the options that considered to be applied as a single batch. Let's combine the 15-month duration of the contract with Romania as an allowed place of performance in this case: 

```
{
  "optionDetails": {
    "optionsToCombine": [
      {
        "id": "",
        "relatedOptions": [
          "1-1",
          "2-2"
        ]
      }
    ]
  }
}
```

### Options for *Requirement*

*Options* could allow to include an applicable *options* into used [requirement](https://github.com/open-contracting-extensions/ocds_requirements_extension). Such an approach allows prescribing more precise ranges of expected values. For example: if CA requires EO to have at least one Google certified specialist on his board but ready to grant EO with additional advantage if such EO has more than one guy:

```
{
  "criteria": [
    {
      "id": "003",
      "title": "Capacity",
      "description": "Minimum qualification requirements",
      "source": "tenderer",
      "relatesTo": "tenderer",
      "requirementGroups": [
        {
          "id": "003-1",
          "requirements": [
            {
              "id": "003-1-1",
              "title": "At least one Google certified specialist on-board",
              "dataType": "boolean",
              "expectedValue": true
            },
            {
              "id": "003-1-2",
              "title": "Number of Google certified staff",
              "description": "",
              "dataType": "integer",
              "minValue": 1,
              "optionDetails": {
                "optionGroups": [
                  {
                    "id": "3",
                    "description": "",
                    "relatesTo": "maxValue",
                    "options": [
                      {
                        "id": "3-1",
                        "maxValue": 4,
                        "minValue":2,
                        "description": "More then one"
                      },
                      {
                        "id": "3-2",
                        "maxValue": 10,
                        "minValue":5,
                        "description": "Up to five specialists"
                      },
                      {
                        "id": "3-3",
                        "minValue": 11,
                        "description": "More then five specialists"
                      }
                    ]
                  }
                ]
              }
            }
          ]
        }
      ]
    },
    {
      "id": "004",
      "title": "Product",
      "description": "Country of origin",
      "source": "tenderer",
      "relatesTo": "item",
      "relatedItem":"1",
      "requirementGroups": [
        {
          "id": "004-1",
          "requirements": [
            {
              "id": "004-1-1",
              "title": "Product has to be manufactured in the EU",
              "dataType": "boolean",
              "expectedValue": true
            },
            {
              "id": "004-1-2",
              "title": "Country of origin",
              "description": "",
              "dataType": "string",
              "optionDetails": {
                "optionGroups": [
                  {
                    "id": "4",
                    "description": "",
                    "options": [
                      {
                        "id": "4-1",
                        "value":"HU",
                        "description": "Hungary"
                      },
                      {
                        "id": "4-2",
                        "value":"IT",
                        "description": "Italy"
                      },
                      {
                        "id": "4-3",
                        "value": "other",
                        "description": "Other EU country"
                      }
                    ]
                  }
                ]
              }
            }
          ]
        }
      ]
    }
  ]
}
```
### Options for *Targets*

*Options* also could be applicable for [*targets*](https://github.com/open-contracting-extensions/ocds_metrics_extension). For example, CA is going to buy 1000 cars during some period but at the same time, he considered to buy less cars during a shorter period. PE can describe *options* for *target*:

```
{
  "targets": [
    {
      "id": "annualNeed",
      "title": "Annual need",
      "description": "The annual need",
      "observations": [
        {
          "period": {
            "startDate": "2015-01-01T00:00:00Z",
            "endDate": "2015-12-31T23:59:59Z"
          },
          "measure": 1000,
          "unit": {
            "id": "NC",
            "name": "car"
          },
          "optionDetails": {
            "optionGroups": [
              {
                "id": "5",
                "description": "",
                "relatesTo": "measure",
                "options": [
                  {
                    "id": "5-1",
                    "measure": 800
                  }
                ]
              },
              {
                "id": "6",
                "description": "",
                "relatesTo": "period",
                "options": [
                  {
                    "id": "6-1",
                    "startDate": "2015-01-01T00:00:00Z",
                    "endDate": "2015-09-31T23:59:59Z"
                  }
                ]
              }
            ],
            "optionsToCombine": [
              {
                "id": "",
                "relatedOptions": [
                "5-1",
                "6-1"
                ]
              }
            ]
          }
        }
      ]
    }
  ]
}
```
