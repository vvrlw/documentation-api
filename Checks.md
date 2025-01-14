# Cloud Conformity Checks API

Below is a list of the available APIs:

- [Create Custom Checks](#create-custom-checks)
- [Update Check](#update-check)
- [List All Checks](#list-all-checks)
- [Get Check Details](#get-check-details)
- [Delete Check](#delete-check)



## Create custom checks
This endpoint is used to create a custom checks. You may pass one check or an array of checks in the JSON body.


**IMPORTANT:**
&nbsp;&nbsp;&nbsp;Some guidelines about using this endpoint:
1. Checks are created as long as your inputs are valid. The onus is on you to ensure the checks you enter are meaningful and useful.
3. Each check object you enter will require a `check.relationships.account`. If you provide an account which you don't have WRITE access to, the check will not be saved.
2. Check Ids are constructed from the parameters entered and follow the format:
   1. **ccc:accountId:ruleId:service:region:resourceId**
   2. If you add a check with the same `accountId`, `ruleId`, `service`, `region`, AND `resourceId` as another existing check in the database, this new check WILL write over the existing check.
   3. Since resource is an optional attribute, checks entered without resource will not have the `resourceId` part of the check Id.

##### Endpoints:

`POST /checks`

##### Parameters
- `data`: a data array (or data object if only creating one check) containing JSONAPI compliant objects with following properties
  - `type`: "checks"
  - `attributes`: An attributes object containing
    - `message`: String, descriptive message about the check
    - `region`: String, a valid AWS region. Please refer to [Cloud Conformity Region Endpoint](https://us-west-2.cloudconformity.com/v1/regions)
    - `resource`: String, the AWS resource this check applies to. (optional)
    - `rule-title`: String, custom rule title. (optional, defaults to "Custom Rule" if not specified).
      - Note: If there are multiple custom checks with the same rule id, then the rule title of the check with the most recently updated date will be used. Hence this field can be used to update a custom rule's title with a new value.

    - `risk-level`: String, one risk level from the following: LOW\| MEDIUM \| HIGH \| VERY_HIGH \| EXTREME
    - `status`: String, SUCCESS or FAILURE
    - `categories`: An array of category (AWS well-architected framework category) strings from the following: security \| cost-optimisation \| reliability \| performance-efficiency  \| operational-excellence (optional)
    - `service`: String, a valid AWS service, please refer to [Cloud Conformity Services Endpoint](https://us-west-2.cloudconformity.com/v1/services)
    - `not-scored`: Boolean, true for informational checks (optional)
    - `tags`: Array, an array of tag strings that follow the format: "key::value". You can enter a max of 20 tags, each tag must not exceed 50 characters. (optional)
    - `resolution-page-url`: Custom defined resolution page url.
    - `extradata`: An array of objects (optional), each object must contain
      - `label`: String, as it will appear on the client UI. Character limit of 20
      - `name`: String, as reference for the back-end. Character limit of 20
      - `type`: String, provide type as you see fit. Character limit of 20
      - `value`: Enter value as you see fit. If entering a number or string, length must not exceed 150.
  - `relationships`: A relationships object containing
    - `account`: An account object containing
      - `data`: A data object containing
        - `id`: String, CloudConformity account id
        - `type`: "accounts"
    - `rule`: An rule object containing
      - `data`: A data object containing
        - `id`: "CUSTOM-001"
        - `type`: "rules"

Example request for creating a check:

```
curl -X POST \
-H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
-d '
{
    "data": [
        {
            "type": "checks",
            "attributes": {
                "extradata": [
                    {
                        "label": "Less than 20 chars",
                        "name": "nameForReference",
                        "type": "META",
                        "value": "string or number or boolean"
                    },
                    {
                        "label": "Less than 20 chars",
                        "name": "forReference",
                        "type": "META",
                        "value": "hello world!"
                    }
                ],
                "message": "Descriptive message about this check",
                "region": "us-west-2",
                "resource": "sg-956d00ea",
                "risk-level": "VERY_HIGH",
                "status": "FAILURE",
                "service": "EC2",
                "categories": ["security"],
                "tags": ["key0::value0", "key1::value1"],
                "resolution-page-url": "http://test.com/custom-001.html"
            },
            "relationships": {
                "account": {
                    "data": {
                        "id": "H19NxM15-",
                        "type": "accounts"
                    }
                },
                "rule": {
                    "data": {
                        "id": "CUSTOM-001",
                        "type": "rules"
                    }
                }
            },
        },
        {
            "attributes": {
                "extradata": [
                    {
                        "label": "Attachments",
                        "name": "Attachments",
                        "type": "META",
                        "value": ""
                    },
                    {
                        "label": "Description",
                        "name": "Description",
                        "type": "META",
                        "value": "default VPC security group"
                    },
                    {
                        "label": "Group Id",
                        "name": "GroupId",
                        "type": "META",
                        "value": "sg-2e885d00"
                    },
                    {
                        "label": "Group Name",
                        "name": "GroupName",
                        "type": "META",
                        "value": "default"
                    },
                    {
                        "label": "Vpc Id",
                        "name": "VpcId",
                        "type": "META",
                        "value": "vpc-c7000fa3"
                    }
                ],
                "message": "Security group default allows ingress from 0.0.0.0/0 to port 53",
                "not-scored": false,
                "region": "us-west-2",
                "resource": "sg-2e885d00",
                "risk-level": "VERY_HIGH",
                "categories": ["security"],
                "service": "EC2",
                "status": "FAILURE"
            },
            "relationships": {
                "account": {
                    "data": {
                        "id": "H19NxM15-",
                        "type": "accounts"
                    }
                },
                "rule": {
                    "data": {
                        "id": "CUSTOM-001",
                        "type": "rules"
                    }
                }
            },
            "type": "checks"
        }
    ]
}' \
https://us-west-2-api.cloudconformity.com/v1/checks
```
Example Response:

```
{
    "data": [
        {
            "type": "checks",
            "id": "ccc:H19NxM15-:CUSTOM-001:EC2:us-west-2:sg-956d00ea",
            "attributes": {
                "region": "us-west-2",
                "status": "FAILURE",
                "risk-level": "VERY_HIGH",
                "pretty-risk-level": "Very High",
                "rule-title": "Custom Rule",
                "message": "Descriptive message about this check",
                "resource": "sg-956d00ea",
                "last-modified-date": 1521660152755,
                "created-date": 1521660152755,
                "failure-discovery-date": 1521660152755,
                "categories": ["security"],
                "resolution-page-url": "http://test.com/custom-001.html",
                "extradata": [
                    {
                        "label": "This will show up on the UI",
                        "name": "nameForReference",
                        "type": "META",
                        "value": "string or number or boolean"
                    },
                    {
                        "label": "It is good to be descriptive",
                        "name": "forReference",
                        "type": "META",
                        "value": "hello world!"
                    }
                ],
                "tags": ["key0::value0", "key1::value1"]
            },
            "relationships": {
                "rule": {
                    "data": {
                        "type": "rules",
                        "id": "CUSTOM-001"
                    }
                },
                "account": {
                    "data": {
                        "type": "accounts",
                        "id": "H19NxM15-"
                    }
                }
            }
        },
        {
            "type": "checks",
            "id": "ccc:H19NxM15-:CUSTOM-001:EC2:us-west-2:sg-2e885d00",
            "attributes": {
                "region": "us-west-2",
                "status": "FAILURE",
                "risk-level": "VERY_HIGH",
                "pretty-risk-level": "Very High",
                "rule-title": "Custom Rule",
                "message": "Security group default allows ingress from 0.0.0.0/0 to port 53",
                "resource": "sg-2e885d48",
                "last-modified-date": 1521660152755,
                "created-date": 1521660152755,
                "failure-discovery-date": 1521660152755,
                "categories": ["security"],
                "extradata": [
                    {
                        "label": "Attachments",
                        "name": "Attachments",
                        "type": "META",
                        "value": ""
                    },
                    {
                        "label": "Description",
                        "name": "Description",
                        "type": "META",
                        "value": "default VPC security group"
                    },
                    {
                        "label": "Group Id",
                        "name": "GroupId",
                        "type": "META",
                        "value": "sg-2e885d00"
                    },
                    {
                        "label": "Group Name",
                        "name": "GroupName",
                        "type": "META",
                        "value": "default"
                    },
                    {
                        "label": "Vpc Id",
                        "name": "VpcId",
                        "type": "META",
                        "value": "vpc-c7000fa3"
                    }
                ]
            },
            "relationships": {
                "rule": {
                    "data": {
                        "type": "rules",
                        "id": "CUSTOM-001"
                    }
                },
                "account": {
                    "data": {
                        "type": "accounts",
                        "id": "H19NxM15-"
                    }
                }
            }
        }
    ]
}
```


## Update check
This endpoint is used to either update one custom check OR suppress/unsuppress one normal check.


**IMPORTANT:**
&nbsp;&nbsp;&nbsp;Some guidelines about using this endpoint:
1. When updating an custom check, you must leave `region`, `resource`, `service` attributes and `relationship.account` and `relationship.rule` unchanged. These are unique identifier parameters for custom checks and must be always present and unchanged once check is created.
2. Suppression of check via this endpoint only works with FAILING checks and not successful checks.

##### Endpoints:

`PATCH /checks/id`

##### Parameters

*The following parameters are for updating a custom check only* 
- `data`: a data object containing JSONAPI compliant object with following properties
  - `type`: "checks"
  - `attributes`: An attributes object containing
    - `message`: String, descriptive message about the check
    - `region`: String, leave UNCHANGED
    - `resource`: String, leave UNCHANGED (optional)
    - `rule-title`: String, custom rule title. (optional, defaults to "Custom Rule" if not specified)
    - `risk-level`: String, one risk level from the following: LOW\| MEDIUM \| HIGH \| VERY_HIGH \| EXTREME
    - `status`: String, SUCCESS or FAILURE
    - `categories`: An array of category (AWS well-architected framework category) strings from the following: security \| cost-optimisation \| reliability \| performance-efficiency  \| operational-excellence (optional)
    - `service`: String, leave UNCHANGED
    - `not-scored`: Boolean, true for informational checks (optional)
    - `tags`: Array, an array of tag strings that follow the format: "key::value". You can enter a max of 20 tags, each tag must not exceed 50 characters. (optional)
    - `resolution-page-url`: Custom defined resolution page url.
    - `extradata`: An array of objects (optional), each object must contain
      - `label`: String, as it will appear on the client UI. Character limit of 20
      - `name`: String, as reference for the back-end. Character limit of 20
      - `type`: String, provide type as you see fit. Character limit of 20
      - `value`: Enter value as you see fit. If entering a number or string, length must not exceed 150.
  - `relationships`: A relationships object containing
    - `account`: An account object containing
      - `data`: A data object containing
        - `id`: String, leave UNCHANGED
        - `type`: "accounts"
    - `rule`: An rule object containing
      - `data`: A data object containing
        - `id`: String, leave UNCHANGED
        - `type`: "rules"


*The following parameters are for normal checks only*
- `data`: a data object containing JSONAPI compliant object with following properties
  - `type`: "checks"
  - `attributes`: An attributes object containing
    - `suppressed`: Boolean, true for suppressing the check
    - `suppressed-until` Number, milliseconds between midnight of January 1, 1970 and the time when you want to suppress the check until. *Null if suppressing indefinitely*
- `meta`: a data object containing
  - `note`: String, a message regarding the reason for this check suppression update.


Example request for updating a custom check:

```
curl -X PATCH \
-H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
-d '
{
    "data": {
        "type": "checks",
        "attributes": {
            "region": "us-west-2",
            "resource": "sg-956d00ea",
            "risk-level": "VERY_HIGH",
            "status": "FAILURE",
            "service": "EC2",
            "categories": ["security"],
            "rule-title": "Custom Rule about EC2 SGs",
            "message": "Updated message about this check",
            "resolution-page-url": "https://test.com/custom-001.html#"
            "extradata": [
                {
                    "label": "This will show up on the UI",
                    "name": "nameForReference",
                    "type": "META",
                    "value": "string or number or boolean"
                },
                {
                    "label": "It is good to be descriptive",
                    "name": "forReference",
                    "type": "META",
                    "value": "hello world!"
                }
            ],
            "tags": ["key0::value0", "key1::value1"]
        },
        "relationships": {
            "rule": {
                "data": {
                    "type": "rules",
                    "id": "CUSTOM-001"
                }
            },
            "account": {
                "data": {
                    "type": "accounts",
                    "id": "H19NxM15-"
                }
            }
        }
    }
}' \
https://us-west-2-api.cloudconformity.com/v1/checks/ccc:H19NxM15-:CUSTOM-001:EC2:us-west-2:sg-956d00ea
```
Example Response:

```
{
    "data": {
        "type": "checks",
        "id": "ccc:H19NxM15-:CUSTOM-001:EC2:us-west-2:sg-956d00ea",
        "attributes": {
            "region": "us-west-2",
            "status": "FAILURE",
            "service": "EC2",
            "risk-level": "VERY_HIGH",
            "pretty-risk-level": "Very High",
            "rule-title": "Custom Rule about EC2 SGs",
            "message": "Updated message about this check",
            "resource": "sg-956d00ea",
            "categories": ["security"],
            "last-modified-date": 1526566995282,
            "created-date": 1521660152755,
            "failure-discovery-date": 1521660152755,
            "resolution-page-url": "https://test.com/custom-001.html#",
            "extradata": [
                {
                    "label": "This will show up on the UI",
                    "name": "nameForReference",
                    "type": "META",
                    "value": "string or number or boolean"
                },
                {
                    "label": "It is good to be descriptive",
                    "name": "forReference",
                    "type": "META",
                    "value": "hello world!"
                }
            ],
            "tags": ["key0::value0", "key1::value1"]
        },
        "relationships": {
            "rule": {
                "data": {
                    "type": "rules",
                    "id": "CUSTOM-001"
                }
            },
            "account": {
                "data": {
                    "type": "accounts",
                    "id": "H19NxM15-"
                }
            }
        }
    }
}
```


Example request for suppressing a check:

```
curl -X PATCH \
-H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
-d '
{
    "data": {
        "type": "checks",
        "attributes": {
            "suppressed": true,
            "suppressed-until": 1526574705655
        }
    },
    "meta": {
        "note": "suppressed for 1 week, failure not-applicable during project xyz"

    }
}; \

https://us-west-2-api.cloudconformity.com/v1/checks/ccc:H19NxM15:EC2-013:EC2:ap-southeast-2:
```
Example Response:

```
{
    "data": {
        "type": "checks",
        "id": "ccc:H19NxM15:Config-001:Config:us-west-2:",
        "attributes": {
            "region": "us-west-2",
            "status": "FAILURE",
            "risk-level": "HIGH",
            "pretty-risk-level": "High",
            "message": "AWS Config is not enabled for us-west-2 region ",
            "last-modified-date": 1526571108174,
            "created-date": 1513472920569,
            "categories": [
                "security"
            ],
            "suppressed": true,
            "last-updated-date": null,
            "failure-discovery-date": 1526571108174,
            "failure-introduced-by": null,
            "resolved-by": null,
            "last-updated-by": null,
            "extradata": null,
            "tags": [],
            "cost": 0,
            "waste": 0,
            "suppressed-until": 1526574705655,
            "not-scored": false,
            "rule-title": "AWS Config Enabled",
            "resolution-page-url": "https://www.cloudconformity.com/conformity-rules/Config/aws-config-enabled.html#"
        },
        "relationships": {
            "rule": {
                "data": {
                    "type": "rules",
                    "id": "Config-001"
                }
            },
            "account": {
                "data": {
                    "type": "accounts",
                    "id": "HJtqfslfG"
                }
            }
        }
    }
}
```


## List All Checks

This endpoint allows you to collect checks for a specified account.

##### Endpoints:

`GET /checks`

##### Parameters
- `accountIds`: A comma-separated list of Cloud Conformity accountIds.
- `page[size]`: Indicates the number of results that should be returned. Maximum value is 1000 and defaults to 100 if not specified
- `page[number]`: Indicates the page number, defaults to 0
- `filter`: Optional parameter including services, regions, categories, statuses, ruleIds, riskLevel, suppressed, and tags.

###### There is a 10k limit to the maximum number of overall results that can be returned. Paging will not work for higher than this limit. To fetch larger numbers, segment your requests using account and region filtering. On larger accounts, filter requests per account, per region, per service.


##### Filtering
The `filter` query parameter is reserved to be used as the basis for filtering.


For example, the following is a request for a page of checks filtered by service `EC2`:

```
GET /checks?accountIds=r1gyR4cqg&page[size]=100&page[number]=0&filter[services]=EC2
```

Multiple filter values can be combined in a comma-separated list. For example the following is a request for a page of checks in `us-west-2` or `us-west-1` regions:
```
GET /checks?accountIds=r1gyR4cqg&page[size]=100&page[number]=0&filter[regions]=us-west-1,us-west-2
```

Furthermore, multiple filters can be applied to a single request. For example, the following is a request to get checks for `us-west-2` region when the status of the check is `SUCCESS`, and it's for `EC2` or `IAM` service in `security` category with `HIGH` risk level
```
GET /checks?accountIds=r1gyR4cqg&page[size]=100&page[number]=0&filter[regions]=us-west-2&filter[statuses]=SUCCESS&filter[categories]=security&filter[riskLevels]=HIGH&filter[services]=EC2,IAM
```


The table below give more information about filter options:

| Name  | Values |
| ------------- | ------------- |
| filter[regions]  | global \| us-east-2 \| us-east-1 \| us-west-1 \| us-west-2 \| ap-south-1 \| ap-northeast-2 \|<br />ap-southeast-1 \| ap-southeast-2 \| ap-northeast-1 \| ca-central-1 \| eu-central-1 \| eu-west-1 \|<br /> eu-west-2 \| sa-east-1 <br /><br />For more information about regions, please refer to [Cloud Conformity Region Endpoint](https://us-west-2.cloudconformity.com/v1/regions) |
| filter[services]  | AutoScaling \| CloudConformity \|CloudFormation \| CloudFront \| CloudTrail \| CloudWatch \|<br />CloudWatchEvents \| CloudWatchLogs \| Config \| DynamoDB \| EBS \| EC2 \| ElastiCache \| Elasticsearch \| ELB \| IAM \| KMS \| RDS \| Redshift \| ResourceGroup \| Route53 \| S3 \| SES \|<br />SNS \| SQS \| VPC \| WAF \| ACM \| Inspector \| TrustedAdvisor \| Shield \| EMR \| Lambda \|<br />Support \| Organizations \| Kinesis \| EFS<br /><br />For more information about services, please refer to [Cloud Conformity Services Endpoint](https://us-west-2.cloudconformity.com/v1/services) |
| filter[categories]  | security \| cost-optimisation \| reliability \| performance-efficiency  \| operational-excellence <br /><br />For more information about categories, please refer to [Cloud Conformity Services Endpoint](https://us-west-2.cloudconformity.com/v1/services) |
| filter[riskLevels]  | LOW\| MEDIUM \| HIGH \| VERY_HIGH \| EXTREME <br /><br />For more information about risk levels, please refer to [Cloud Conformity Services Endpoint](https://us-west-2.cloudconformity.com/v1/services) |
| filter[statuses]  | SUCCESS \| FAILURE |
| filter[ruleIds]  | EC2-001 \| EC2-002 \| etc <br /><br />For more information about rules, please refer to [Cloud Conformity Services Endpoint](https://us-west-2.cloudconformity.com/v1/services) |
| filter[suppressedFilterMode]  | "v1" \| "v2" <br /><br /> Whether to use the `"v1"` or `"v2"` suppressed functionality. `"v1"`: Using `suppressed=true` will return both suppressed and unsuppressed checks, `suppressed=false` will just return unsuppressed checks. `"v2"`: Using `suppressed=true` return will just return suppressed checks. Defaults to "v1".
| filter[suppressed]  | true \| false <br /><br /> Whether or not include all suppressed checks. The default value is `true` for "v1" and omitted for "v2" |
| filter[createdDate]  | The date when the check was created<br /><br />The numeric value of the specified date as the number of milliseconds since January 1, 1970, 00:00:00 UTC |
| filter[tags]  | Any assigned metadata tags to your AWS resources |

<br />



Example Request:

```
curl -H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
https://us-west-2-api.cloudconformity.com/v1/checks?accountIds=r1gyR4cqg&page[size]=100&page[number]=0&filter[regions]=us-west-2&filter[ruleIds]=EC2-001,EC2-002&filter[statuses]=SUCCESS&filter[categories]=security&filter[riskLevels]=HIGH&filter[services]=EC2&filter[createdDate]=1502572157914
```
Example Response:
###### Note the size of this response can be quite large and the example below is purposefully truncated

```
{
    "data": [
        {
            "type": "checks",
            "id": "ccc:94qQ4DnOB:AG-003:APIGateway:us-east-1:pl63negesk",
            "attributes": {
                "region": "us-east-1",
                "status": "FAILURE",
                "risk-level": "LOW",
                "pretty-risk-level": "Low",
                "message": "API Security Automations - WAF Bad Bot API has stages without active tracing enabled",
                "resource": "pl63negesk",
                "descriptorType": "apigateway-restapi",
                "resourceName": "API Gateway REST API",
                "last-modified-date": 1561697456359,
                "created-date": 1561697456359,
                "categories": [
                    "operational-excellence"
                ],
                "last-updated-date": null,
                "failure-discovery-date": 1561697456359,
                "ccrn": "ccrn:aws:94qQ4DnOB:APIGateway:us-east-1:pl63negesk",
                "tags": [],
                "cost": 0,
                "waste": 0,
                "rule-title": "Tracing Enabled",
                "link": "https://us-east-1.console.aws.amazon.com/apigateway/home?region=us-east-1#/apis/pl63negesk/resources",
                "provider": "aws",
                "resolutionPageUrl": "https://www.cloudconformity.com/conformity-rules/APIGateway/tracing.html#B1nHYYpwx"
            },
            "relationships": {
                "rule": {
                    "data": {
                        "type": "rules",
                        "id": "AG-003"
                    }
                },
                "account": {
                    "data": {
                        "type": "accounts",
                        "id": "94qQ4DnOB"
                    }
                }
            }
        }

    ],
    "meta": {
        "total-pages": 714
    }
}
```


## Get Check Details

This endpoint allows you to get the details of the specified check.

##### Endpoints:

`GET /checks/id`

##### Parameters
- `id`: The Cloud Conformity ID of the check


Example Request:

```
curl -H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
https://us-west-2-api.cloudconformity.com/v1/checks/ccc:r2gyR4cqg:IAM-017:IAM:global:groups-test
```
Example Response:
```
{
    "data": {
        "type": "checks",
        "id": "ccc:r2gyR4cqg:IAM-017:IAM:global:groups-test",
        "attributes": {
            "region": "global",
            "status": "FAILURE",
            "risk-level": "LOW",
            "pretty-risk-level": "High",
            "message": "IAM Group test contains no user",
            "last-modified-date": 1500166639466,
            "created-date": 1500166639466
            "last-updated-date": 1500166639466,
            "failure-discovery-date": 1498910777689,
            "last-updated-by": "SYSTEM",
            "resolved-date": 1518409298274,
            "resolved-by": null,
            "ccrn": "ccrn:aws:r1gyR4cqg:IAM:global:groups-test",
            "extradata": null,
            "tags": [],
            "cost": 0,
            "waste": 0,
            "not-scored": false,
            "ignored": null,
            "rule-title": "Password Policy Present",
            "resolution-page-url": "https://www.cloudconformity.com/conformity-rules/IAM/unused-iam-group.html#"
        },
        "relationships": {
            "rule": {
                "data": {
                    "type": "rules",
                    "id": "IAM-017"
                }
            },
            "account": {
                "data": {
                    "type": "accounts",
                    "id": "r1gyR4cqg"
                }
            }
        }
    }
}
```





## Delete check

A DELETE request to this endpoint allows a user with WRITE access to the associated account to delete the check.

##### Endpoints:

`DELETE /checks/checkId`

Example Request:
```
curl -X DELETE \
-H "Content-Type: application/vnd.api+json" \
-H "Authorization: ApiKey S1YnrbQuWagQS0MvbSchNHDO73XHqdAqH52RxEPGAggOYiXTxrwPfmiTNqQkTq3p" \
https://us-west-2-api.cloudconformity.com/v1/checks/ccc:H19NxM15-:CUSTOM-001:EC2:us-west-2:sg-956d00ea
```

Example Response:

```
{
    "meta": [{
        "status": "success",
        "message": "Check successfully deleted"
    }]
}
```


