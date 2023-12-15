# 1. Implementation Guide

This document contains the REQUIRED and RECOMMENDED standard functionality that must be implemented by any Mobility Service Provider Platform a.k.a BPPs and Mobility Consumer Platform a.k.a BAPs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## 1.1 Discovery of Mobility providers

### 1.1.1 Recommendations for BPPs
The following recommendations need to be considered when implementing discovery functionality for a Mobility BPP

- REQUIRED. The BPP MUST implement the `search` endpoint to receive an `Intent` object sent by BAPs
- REQUIRED. The BPP MUST return a catalog of Mobility products on the `on_search` callback endpoint specified in the `context.bap_uri` field of the `search` request body.
- REQUIRED. The BPP MUST map its Mobility products to the `Item` schema.
- REQUIRED. Any Mobility service provider-related information like name, logo, short description must be mapped to the `Provider.descriptor` schema
- REQUIRED. Any form that must be filled before receiving a quotation must be mapped to the `XInput` schema
- REQUIRED. If the platform wants to group its products under a specific category, it must map each category to the `Category` schema
- REQUIRED. Any service fulfillment-related information MUST be mapped to the `Fulfillment` schema.
- REQUIRED. If the BPP does not want to respond to a search request, it MUST return a `ack.status` value equal to `NACK`
- RECOMMENDED. Upon receiving a `search` request, the BPP SHOULD return a catalog that best matches the intent. This can be done by indexing the catalog against the various probable paths in the `Intent` schema relevant to typical Mobility use cases

### 1.1.2 Recommendations for BAPs
- REQUIRED. The BAP MUST call the `search` endpoint of the BG to discover multiple BPPs on a network
- REQUIRED. The BAP MUST implement the `on_search` endpoint to consume the `Catalog` objects containing Mobility Products sent by BPPs.
- REQUIRED. The BAP MUST expect multiple catalogs sent by the respective Mobility Providers on the network
- REQUIRED. The Mobility products can be found in the `Catalog.providers[].items[]` array in the `on_search` request
- REQUIRED. If the `catalog.providers[].items[].xinput` object is present, then the BAP MUST redirect the user to, or natively render the form present on the link specified on the `items[].xinput.form.url` field.
- REQUIRED. If the `catalog.providers[].items[].xinput.required` field is set to `"true"` , then the BAP MUST NOT fire a `select`, `init` or `confirm` call until the form is submitted and a successful response is received
- RECOMMENDED. If the `catalog.providers[].items[].xinput.required` field is set to `"false"` , then the BAP SHOULD allow the user to skip filling the form


### Example
A search request for a Mobility  may look like this
```
{
  "context": {
    "country": "IND",
    "domain": "nic2004:60221",
    "timestamp": "2023-03-23T04:41:16Z",
    "bap_id": "example-bap.com",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "city": "std:080",
    "core_version": "0.9.4",
    "action": "search",
    "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
  },
  "message": {
    "intent": {
      "fulfillment": {
        "start": {
          "location": {
            "gps": "12.923608703179461, 77.61462964117527"
          }
        },
        "end": {
          "location": {
            "gps": "12.9346302, 77.61533969999999"
          }
        },
        "customer": {
          "person": {
            "tags": {
              "groups/1/descriptor/name": "Localization",
              "groups/1/display": false,
              "groups/1/list/1/descriptor/name": "Language",
              "groups/1/list/1/value": "en"
            }
          }
        }
      }
    }
  }
}

```

An example catalog of a Mobility  service may look like this
```
{
  "message": {
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:43:02Z",
      "bap_id": "example-bap.com",
      "transaction_id": "870782be-6757-43f1-945c-8eeaf9536259",
      "bpp_id": "example-bpp.com",
      "message_id": "21e54d3c-9c3b-47c1-aa3b-b0e7b20818ee",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_search",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "catalog": {
      "bpp/descriptor": {
        "name": "InstaAuto"
      },
      "bpp/providers": [
        {
          "locations": [
            {
              "id": "1",
              "gps": "12.9164682,77.6089985"
            },
            {
              "id": "2",
              "gps": "12.91671,77.6092983"
            },
            {
              "id": "3",
              "gps": "12.9165733,77.6152167"
            },
            {
              "id": "4",
              "gps": "12.9068578,77.6044567"
            }
          ],
          "items": [
            {
              "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
              "descriptor": {
                "name": "Auto Ride",
                "code": "RIDE"
              },
              "price": {
                "maximum_value": "156",
                "currency": "INR",
                "minimum_value": "176",
                "value": "156 - 176 INR"
              },
              "tags": {
                "groups/1/descriptor/name": "Daytime Charges",
                "groups/1/display": true,
                "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
                "groups/1/list/1/value": "₹ 30 upto 2 km",
                "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
                "groups/1/list/2/value": "₹15 / km",
                "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
                "groups/1/list/3/value": "₹ 10",
                "groups/1/list/4/descriptor/name": "Nominal Fare",
                "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
                "groups/1/list/4/value": "₹ 10",
                "groups/1/list/5/descriptor/name": "Waiting Charges",
                "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
                "groups/1/list/5/value": "₹ 0 / min",
                "groups/2/descriptor/name": "Night Charges",
                "groups/2/display": true,
                "groups/2/list/1/descriptor/name": "Night Charges",
                "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
                "groups/2/list/2/descriptor/name": "Night Shift Start",
                "groups/2/list/2/value": "22:00:00",
                "groups/2/list/3/descriptor/name": "Night Shift End",
                "groups/2/list/3/value": "05:00:00",
                "groups/3/descriptor/name": "General Information",
                "groups/3/display": true,
                "groups/3/list/1/descriptor/name": "Distance to nearest driver",
                "groups/3/list/1/value": "661 m",
                "groups/3/list/2/descriptor/name": "Wait time upto",
                "groups/3/list/2/value": "3 min"
              },
              "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
              "payment_id": "1"
            }
          ],
          "fulfillments": [
            {
              "id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
              "start": {
                "location": {
                  "gps": "12.9099828, 77.6118226",
                  "address": {
                    "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                    "country": "India",
                    "building": "6th Main Rd",
                    "state": "Karnataka 560061",
                    "city": "Bengaluru",
                    "locality": "Uttarahalli Hobli",
                    "door": "98A, Sarovarm 2nd cross",
                    "area_code": "560061",
                    "street": "Ramanjaneyanagar"
                  }
                }
              },
              "end": {
                "location": {
                  "gps": "12.9351856, 77.62459969999999",
                  "address": {
                    "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                    "country": "India",
                    "building": "Rangadore Memorial Hospital",
                    "state": "Karnataka",
                    "city": "Bengaluru",
                    "locality": "Basavanagudi",
                    "door": null,
                    "area_code": "",
                    "street": "Chikkanna Garden"
                  }
                }
              },
              "vehicle": {
                "category": "AUTO_RICKSHAW"
              }
            }
          ],
          "payments": [
            {
              "id": "1",
              "type": "ON-FULFILLMENT",
              "collected_by": "BPP"
            }
          ]
        }
      ]
    }
  }
}
```

## 1.2 Application for local-Mobility 
This section provides recommendations for implementing the APIs related to a Mobility.

### 1.2.1 Recommendations for BPPs

#### 1.2.1.1 Selecting a Mobility service from the catalog
- REQUIRED. The BPP MUST implement the `select` endpoint on the url specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST check for a form submission at the URL specified on the `xinput.form.url` before acknowledging a `select` request.
- REQUIRED. If the Mobility provider has successfully received the information submitted by the Mobility consumer, the BPP must return an acknowledgement with `ack.status` set to `ACK` in response to the `select` request
- REQUIRED. If the Mobility provider has returned a successful acknowledgement to a `select` request, it MUST send the offer encapsulated in an `Order` object

#### 1.2.1.2 Initializing a Mobility service
- REQUIRED. The BPP MUST implement the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.1.3 Confirming the Mobility service
- REQUIRED. The BPP MUST implement the `confirm` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.2 Recommendations for BAPs

#### 1.2.1.1 Selecting a Mobility service from the catalog
- REQUIRED. The BAP user MUST submit the form on the url received from  `on_search`  under `xinput.form.url`.
- REQUIRED. The BAP MUST implement the `on_select` endpoint on the url specified in the `context.bap_uri` field sent during `select`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.2.2  Initializing a Mobility service
- REQUIRED. The BAP MUST hit the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_init` endpoint on the url specified in  the `context.bap_uri` field sent during `init`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.2.3 Confirming the Mobility service
- REQUIRED. The BAP MUST hit the `confirm` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_confirm` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `confirm`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.3 Example Workflow

### 1.2.3 Example Requests

Below is an example of a `select` request
```
{
  "context": {
    "country": "IND",
    "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
    "domain": "nic2004:60221",
    "timestamp": "2023-03-23T04:46:45Z",
    "bap_id": "example-bap.com",
    "transaction_id": "870782be-6757-43f1-945c-8eeaf9536259",
    "bpp_id": "example-bpp.com",
    "message_id": "432fdfd6-0457-47b6-9fac-80cbe5c0a75b",
    "city": "std:080",
    "core_version": "0.9.4",
    "action": "select",
    "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1",
    "max_callbacks": 3,
    "ttl": "P120S"
  },
  "message": {
    "order": {
      "items": [
        {
          "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e"
        }
      ],
      "fulfillment": {
        "start": {
          "location": {
            "gps": "12.910458, 77.543089",
            "address": {
              "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
              "country": "India",
              "building": "6th Main Rd",
              "state": "Karnataka 560061",
              "city": "Bengaluru",
              "locality": "Uttarahalli Hobli",
              "door": "98A, Sarovarm 2nd cross",
              "area_code": "560061",
              "street": "Ramanjaneyanagar"
            }
          }
        },
        "end": {
          "location": {
            "gps": "12.9535139, 77.5710434",
            "address": {
              "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
              "country": "India",
              "building": "Rangadore Memorial Hospital",
              "state": "Karnataka",
              "city": "Bengaluru",
              "locality": "Basavanagudi",
              "door": null,
              "area_code": "",
              "street": "Chikkanna Garden"
            }
          }
        },
        "customer": {
          "person": {
            "tags": {
              "groups/1/descriptor/name": "Localization",
              "groups/1/display": false,
              "groups/1/list/1/descriptor/name": "Language",
              "groups/1/list/1/value": "en"
            }
          }
        }
      }
    }
  }
}
```

Below is an example of an `on_select` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "transaction_id": "870782be-6757-43f1-945c-8eeaf9536259",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_select",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "provider": {
          "id": "e8542642-0f4a-454c-9a9f-f46110c367a3",
          "descriptor": {
            "name": "Raghavendra J"
          }
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "descriptor": {
              "name": "Auto Ride",
              "code": "RIDE"
            },
            "tags": {
              "groups/1/descriptor/name": "Daytime Charges",
              "groups/1/display": true,
              "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
              "groups/1/list/1/value": "₹ 30 upto 2 km",
              "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
              "groups/1/list/2/value": "₹15 / km",
              "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
              "groups/1/list/3/value": "₹ 10",
              "groups/1/list/4/descriptor/name": "Nominal Fare",
              "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/4/value": "₹ 10",
              "groups/1/list/5/descriptor/name": "Nominal Fare",
              "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/5/value": "₹ 0 / min",
              "groups/2/descriptor/name": "Waiting Charges",
              "groups/2/display": true,
              "groups/2/list/1/descriptor/name": "Night Charges",
              "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
              "groups/2/list/2/descriptor/name": "Night Shift Start",
              "groups/2/list/2/value": "22:00:00",
              "groups/2/list/3/descriptor/name": "Night Shift End",
              "groups/2/list/3/value": "05:00:00",
              "groups/3/descriptor/name": "General Information",
              "groups/3/display": true,
              "groups/3/list/1/descriptor/name": "Distance to nearest driver",
              "groups/3/list/1/value": "661 m",
              "groups/3/list/2/descriptor/name": "Wait time upto",
              "groups/3/list/2/value": "3 min"
            },
            "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
            "payment_id": "1"
          }
        ],
        "quote": {
          "value": "76",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            }
          ],
          "ttl": "P200S"
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "state": {
            "descriptor": {
              "name": "Found drivers",
              "code": "AGENTS_FOUND"
            }
          },
          "start": {
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW"
          }
        }
      }
    }
  }
  
```


Below is an example of a `init` request
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "init",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "provider": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f"
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "fulfillment_id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
            "payment_id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f"
          }
        ],
        "quote": {
          "value": "76",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            }
          ]
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "start": {
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW"
          }
        },
        "payment": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
          "type": "ON-FULFILLMENT",
          "collected_by": "BPP"
        },
        "customer": {
          "person": {
            "name": "John Doe",
            "phone": "+91-9897867564",
            "tags": {
              "groups/1/descriptor/name": "Localization",
              "groups/1/display": false,
              "groups/1/list/1/descriptor/name": "Language",
              "groups/1/list/1/value": "en"
            }
          }
        }
      }
    }
  }
  
```

Below is an example of an `on_init` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_init",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68",
        "provider": {
          "id": "e8542642-0f4a-454c-9a9f-f46110c367a3",
          "descriptor": {
            "name": "Raghavendra J"
          }
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "descriptor": {
              "name": "Auto Ride",
              "code": "RIDE"
            },
            "tags": {
              "groups/1/descriptor/name": "Daytime Charges",
              "groups/1/display": true,
              "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
              "groups/1/list/1/value": "₹ 30 upto 2 km",
              "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
              "groups/1/list/2/value": "₹15 / km",
              "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
              "groups/1/list/3/value": "₹ 10",
              "groups/1/list/4/descriptor/name": "Nominal Fare",
              "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/4/value": "₹ 10",
              "groups/1/list/5/descriptor/name": "Nominal Fare",
              "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/5/value": "₹ 0 / min",
              "groups/2/descriptor/name": "Waiting Charges",
              "groups/2/display": true,
              "groups/2/list/1/descriptor/name": "Night Charges",
              "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
              "groups/2/list/2/descriptor/name": "Night Shift Start",
              "groups/2/list/2/value": "22:00:00",
              "groups/2/list/3/descriptor/name": "Night Shift End",
              "groups/2/list/3/value": "05:00:00",
              "groups/3/descriptor/name": "General Information",
              "groups/3/display": true,
              "groups/3/list/1/descriptor/name": "Distance to nearest driver",
              "groups/3/list/1/value": "661 m",
              "groups/3/list/2/descriptor/name": "Wait time upto",
              "groups/3/list/2/value": "3 min"
            },
            "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
            "payment_id": "1"
          }
        ],
        "quote": {
          "value": "81",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            },
            {
              "title": "CGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            },
            {
              "title": "SGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            }
          ]
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "start": {
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW"
          },
          "customer": {
            "person": {
              "name": "John Doe",
              "phone": "+91-9897867564",
              "tags": {
                "groups/1/descriptor/name": "Localization",
                "groups/1/display": false,
                "groups/1/list/1/descriptor/name": "Language",
                "groups/1/list/1/value": "en"
              }
            }
          }
        },
        "payment": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
          "type": "ON-FULFILLMENT",
          "params": {
            "amount": "81",
            "currency": "INR",
            "transaction_status": "NOT-PAID"
          }
        }
      }
    }
  }
```

Below is an example of a `confirm` request
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "confirm",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68",
        "provider": {
          "id": "e8542642-0f4a-454c-9a9f-f46110c367a3",
          "descriptor": {
            "name": "Raghavendra J"
          }
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "descriptor": {
              "name": "Auto Ride",
              "code": "RIDE"
            },
            "tags": {
              "groups/1/descriptor/name": "Daytime Charges",
              "groups/1/display": true,
              "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
              "groups/1/list/1/value": "₹ 30 upto 2 km",
              "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
              "groups/1/list/2/value": "₹15 / km",
              "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
              "groups/1/list/3/value": "₹ 10",
              "groups/1/list/4/descriptor/name": "Nominal Fare",
              "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/4/value": "₹ 10",
              "groups/1/list/5/descriptor/name": "Nominal Fare",
              "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/5/value": "₹ 0 / min",
              "groups/2/descriptor/name": "Waiting Charges",
              "groups/2/display": true,
              "groups/2/list/1/descriptor/name": "Night Charges",
              "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
              "groups/2/list/2/descriptor/name": "Night Shift Start",
              "groups/2/list/2/value": "22:00:00",
              "groups/2/list/3/descriptor/name": "Night Shift End",
              "groups/2/list/3/value": "05:00:00",
              "groups/3/descriptor/name": "General Information",
              "groups/3/display": true,
              "groups/3/list/1/descriptor/name": "Distance to nearest driver",
              "groups/3/list/1/value": "661 m",
              "groups/3/list/2/descriptor/name": "Wait time upto",
              "groups/3/list/2/value": "3 min"
            },
            "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
            "payment_id": "1"
          }
        ],
        "quote": {
          "value": "81",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            },
            {
              "title": "CGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            },
            {
              "title": "SGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            }
          ]
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "start": {
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW"
          },
          "customer": {
            "person": {
              "name": "John Doe",
              "phone": "+91-9897867564",
              "tags": {
                "groups/1/descriptor/name": "Localization",
                "groups/1/display": false,
                "groups/1/list/1/descriptor/name": "Language",
                "groups/1/list/1/value": "en"
              }
            }
          }
        },
        "payment": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
          "type": "ON-FULFILLMENT",
          "params": {
            "amount": "81",
            "currency": "INR",
            "transaction_status": "NOT-PAID"
          }
        }
      }
    }
  }
```
Below is an example of an `on_confirm` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_confirm",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68",
        "provider": {
          "id": "e8542642-0f4a-454c-9a9f-f46110c367a3",
          "descriptor": {
            "name": "Raghavendra J"
          }
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "descriptor": {
              "name": "Auto Ride",
              "code": "RIDE"
            },
            "tags": {
              "groups/1/descriptor/name": "Daytime Charges",
              "groups/1/display": true,
              "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
              "groups/1/list/1/value": "₹ 30 upto 2 km",
              "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
              "groups/1/list/2/value": "₹15 / km",
              "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
              "groups/1/list/3/value": "₹ 10",
              "groups/1/list/4/descriptor/name": "Nominal Fare",
              "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/4/value": "₹ 10",
              "groups/1/list/5/descriptor/name": "Nominal Fare",
              "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/5/value": "₹ 0 / min",
              "groups/2/descriptor/name": "Waiting Charges",
              "groups/2/display": true,
              "groups/2/list/1/descriptor/name": "Night Charges",
              "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
              "groups/2/list/2/descriptor/name": "Night Shift Start",
              "groups/2/list/2/value": "22:00:00",
              "groups/2/list/3/descriptor/name": "Night Shift End",
              "groups/2/list/3/value": "05:00:00",
              "groups/3/descriptor/name": "General Information",
              "groups/3/display": true,
              "groups/3/list/1/descriptor/name": "Distance to nearest driver",
              "groups/3/list/1/value": "661 m",
              "groups/3/list/2/descriptor/name": "Wait time upto",
              "groups/3/list/2/value": "3 min"
            },
            "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
            "payment_id": "1"
          }
        ],
        "quote": {
          "value": "81",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            },
            {
              "title": "CGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            },
            {
              "title": "SGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            }
          ]
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "state": {
            "descriptor": {
              "code": "DRIVER_EN_ROUTE",
              "name": "Driver is on the way"
            }
          },
          "start": {
            "authorization": {
              "type": "OTP",
              "token": "234234"
            },
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "phone": "+91-98978675645",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW",
            "registration": "KA01JG1231"
          },
          "customer": {
            "person": {
              "name": "John Doe",
              "phone": "+91-9897867564",
              "tags": {
                "groups/1/descriptor/name": "Localization",
                "groups/1/display": false,
                "groups/1/list/1/descriptor/name": "Language",
                "groups/1/list/1/value": "en"
              }
            }
          }
        },
        "payment": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
          "type": "ON-FULFILLMENT",
          "params": {
            "amount": "81",
            "currency": "INR",
            "transaction_status": "NOT-PAID"
          }
        }
      }
    }
  }
  
```

## 1.3 Fulfillment of Mobility Services
This section contains recommendations for implementing the APIs related to fulfilling a Mobility service

### 1.3.1 Recommendations for BPPs

#### 1.3.1.1 Sending status updates
- REQUIRED. The BPP MUST implement the `status` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.2 Updating a Mobility service
- REQUIRED. The BPP MUST implement the `update` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.3 Cancelling a Mobility service
- REQUIRED. The BPP MUST implement the `cancel` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_cancellation_reasons` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.4 Real-time tracking
- REQUIRED. The BPP MUST implement the `track` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.2 Recommendations for BAPs

#### 1.3.2.1 Receiving status updates
- REQUIRED. The BAP MUST implement the `on_status` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `status`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.2 Updating a Mobility services
- REQUIRED. The BAP MUST implement the `on_update` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `update`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.3 Cancelling a Mobility services
- REQUIRED. The BAP MUST implement the `on_cancel` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `cancel`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `cancellation_reasons` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_cancellation_reasons`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.4 Real-time tracking
- REQUIRED. The BAP MUST implement the `on_track` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `track`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.3 Example Workflow

### 1.3.4 Example Requests

Below is an example of a `status` request
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "status",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order_id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68"
    }
  }
  
```

Below is an example of an `on_status` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "transaction_id": "b580c989-f84d-4abe-af28-2c818aafce3b",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_status",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "order": {
        "id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68",
        "provider": {
          "id": "e8542642-0f4a-454c-9a9f-f46110c367a3",
          "descriptor": {
            "name": "Raghavendra J"
          }
        },
        "items": [
          {
            "id": "5777a0bf-9a08-49aa-a97d-1e5561a9622e",
            "descriptor": {
              "name": "Auto Ride",
              "code": "RIDE"
            },
            "tags": {
              "groups/1/descriptor/name": "Daytime Charges",
              "groups/1/display": true,
              "groups/1/list/1/descriptor/name": "Min Fare upto 2 km",
              "groups/1/list/1/value": "₹ 30 upto 2 km",
              "groups/1/list/2/descriptor/name": "Rate above Min. Fare",
              "groups/1/list/2/value": "₹15 / km",
              "groups/1/list/3/descriptor/name": "Driver Pickup Charges",
              "groups/1/list/3/value": "₹ 10",
              "groups/1/list/4/descriptor/name": "Nominal Fare",
              "groups/1/list/4/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/4/value": "₹ 10",
              "groups/1/list/5/descriptor/name": "Nominal Fare",
              "groups/1/list/5/descriptor/short_desc": "Driver may quote extra to cover for traffic, chance of return trip, etc.",
              "groups/1/list/5/value": "₹ 0 / min",
              "groups/2/descriptor/name": "Waiting Charges",
              "groups/2/display": true,
              "groups/2/list/1/descriptor/name": "Night Charges",
              "groups/2/list/1/value": "1.5x of daytime charges applicable at night from 10 PM to 5 PM",
              "groups/2/list/2/descriptor/name": "Night Shift Start",
              "groups/2/list/2/value": "22:00:00",
              "groups/2/list/3/descriptor/name": "Night Shift End",
              "groups/2/list/3/value": "05:00:00",
              "groups/3/descriptor/name": "General Information",
              "groups/3/display": true,
              "groups/3/list/1/descriptor/name": "Distance to nearest driver",
              "groups/3/list/1/value": "661 m",
              "groups/3/list/2/descriptor/name": "Wait time upto",
              "groups/3/list/2/value": "3 min"
            },
            "fulfillment_id": "fb5c84d4-1b59-4b9d-96b5-9d79107432c5",
            "payment_id": "1"
          }
        ],
        "quote": {
          "value": "81",
          "currency": "INR",
          "breakup": [
            {
              "title": "Base Fare",
              "price": {
                "value": "30",
                "currency": "INR"
              }
            },
            {
              "title": "Per km fare",
              "price": {
                "value": "56",
                "currency": "INR"
              }
            },
            {
              "title": "CGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            },
            {
              "title": "SGST @ 5%",
              "price": {
                "value": "2.5",
                "currency": "INR"
              }
            }
          ]
        },
        "fulfillment": {
          "id": "fulf_5cf064d5-4c0a-42d3-b73d-5f19a6f7468e",
          "state": {
            "descriptor": {
              "code": "RIDE_STARTED",
              "name": "Your ride has started"
            }
          },
          "start": {
            "authorization": {
              "type": "OTP",
              "token": "234234"
            },
            "location": {
              "gps": "13.008935, 77.6444085",
              "address": {
                "ward": "Uttarahalli Hobli, Ramanjaneyanagar",
                "country": "India",
                "building": "6th Main Rd",
                "state": "Karnataka 560061",
                "city": "Bengaluru",
                "locality": "Uttarahalli Hobli",
                "door": "98A, Sarovarm 2nd cross",
                "area_code": "560061",
                "street": "Ramanjaneyanagar"
              }
            }
          },
          "end": {
            "location": {
              "gps": "12.9711869, 77.5868122",
              "address": {
                "ward": "Basavanagudi, Chikkanna Garden, Rangadore Memorial Hospital",
                "country": "India",
                "building": "Rangadore Memorial Hospital",
                "state": "Karnataka",
                "city": "Bengaluru",
                "locality": "Basavanagudi",
                "door": null,
                "area_code": "",
                "street": "Chikkanna Garden"
              }
            }
          },
          "agent": {
            "name": "RAGHAVENDRA J",
            "phone": "+91-98978675645",
            "rateable": true,
            "rating": "5"
          },
          "vehicle": {
            "category": "AUTO_RICKSHAW",
            "registration": "KA01JG1231"
          },
          "customer": {
            "person": {
              "name": "John Doe",
              "phone": "+91-9897867564",
              "tags": {
                "groups/1/descriptor/name": "Localization",
                "groups/1/display": false,
                "groups/1/list/1/descriptor/name": "Language",
                "groups/1/list/1/value": "en"
              }
            }
          }
        },
        "payment": {
          "id": "7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
          "type": "ON-FULFILLMENT",
          "params": {
            "amount": "81",
            "currency": "INR",
            "transaction_status": "NOT-PAID"
          }
        }
      }
    }
  }
  
```

## 1.4 Post-fulfillment of Mobility Services
This section contains recommendations for implementing the APIs after fulfilling a Mobility service

### 1.4.1 Recommendations for BPPs

#### 1.4.1.1 Rating and Feedback
- REQUIRED. The BPP MUST implement the `rating` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_rating_categories` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.1.2 Providing Support
- REQUIRED. The BPP MUST implement the `support` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.2 Recommendations for BAPs

#### 1.4.2.1 Rating and Feedback
- REQUIRED. The BAP MUST implement the `on_rating` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `rating`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `rating_categories` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_rating_categories`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.2.2 Providing Support
- REQUIRED. The BAP MUST implement the `on_support` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `support`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.3 Example Workflow

### 1.4.4 Example Requests

Below is an example of a `get_rating_categories` request
```

```

Below is an example of an `rating_categories` callback
```

```

Below is an example of a `rating` request

```
{
  "context": {
    "country": "IND",
    "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
    "domain": "nic2004:60221",
    "timestamp": "2023-03-23T04:46:45Z",
    "bap_id": "example-bap.com",
    "transaction_id": "870782be-6757-43f1-945c-8eeaf9536259",
    "bpp_id": "example-bpp.com",
    "message_id": "432fdfd6-0457-47b6-9fac-80cbe5c0a75b",
    "city": "std:080",
    "core_version": "0.9.4",
    "action": "rating",
    "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
  },
  "message": {
    "id": "b0462745-f6c9-4100-bbe7-4fa3648b6b40",
    "rating_category": "DRIVER",
    "value": 4,
    "feedback_form": [
      {
        "id": "1",
        "answer": "Driver took me on a route not shown by Google Maps and we reached 10 minutes before time"
      }
    ]
  }
}
```

Below is an example of an `on_rating` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T05:41:15Z",
      "bap_id": "api.beckn.juspay.in/pilot/bap/cab/v1",
      "bpp_id": "api.beckn.juspay.in/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "message_id": "2a17e268-1dc4-4d1a-98a2-17554a50c7d2",
      "city": "Bangalore",
      "core_version": "0.9.4",
      "action": "on_rating",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "feedback_ack": true,
      "rating_ack": true
    }
  }
```

Below is an example of a `support` request
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T04:48:53Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "message_id": "8926b747-0362-4fcc-b795-0994a6287700",
      "transaction_id": "870782be-6757-43f1-945c-8eeaf9536259",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "support",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "ref_id": "7751bd26-3fdc-47ca-9b64-e998dc5abe68"
    }
  }
```

Below is an example of an `on_support` callback
```
{
    "context": {
      "country": "IND",
      "bpp_uri": "https://api.example-bpp.com/dobpp/beckn/7f7896dd-787e-4a0b-8675-e9e6fe93bb8f",
      "domain": "nic2004:60221",
      "timestamp": "2023-03-23T05:41:09Z",
      "bap_id": "example-bap.com",
      "bpp_id": "example-bpp.com",
      "message_id": "ec3dea8c-c64c-4f06-b2a0-ec1f9584d7ba",
      "city": "std:080",
      "core_version": "0.9.4",
      "action": "on_support",
      "bap_uri": "https://api.example-bap.com/pilot/bap/cab/v1"
    },
    "message": {
      "phone": "+918068870525",
      "email": "support@example-bpp.com",
      "url": "https://support.example-bpp.com/gethelp"
    }
  }
```
