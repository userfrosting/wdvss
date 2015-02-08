# Web Data Validation and Sanitization Standard (WDVSS)

## Overview

The goal of this project is to create an interoperable standard for validating and sanitizing data exchanged over the web.  In particular, it focuses on data submitted to a web server via HTTP GET and POST requests.  It should be flexible enough to accommodate both server-side and client-side validation.

## Background

Validating and sanitizing client data is an essential part of modern web applications and services.  Proper server-side sanitization and validation is critical to server security, protecting web servers and their users from SQL injection, cross-site scripting (XSS), and other malicious attacks.  Validation is also an important aspect of user experience, communicating and enforcing the requirements of the underlying data model.  It is suprising, then, that no standard currently exists for how this should be done.

## Scope

Although the examples used in this document will be presented in the [JavaScript Object Notation (JSON)](http://www.json.org/) format, the standard itself is not meant to be tied to any particular data format.  The standard shall be confined to validating and sanitizing HTTP GET and POST requests - other types of requests and communication protocols such as FTP, POP, etc are not addressed by this standard.

## The Standard

### Schema

A **schema** is a document that partially or fully specifies the rules for validating and sanitizing data submitted by a client to a server as part of a single HTTP POST or GET request.

A schema shall consist of a collection of **fields**.  Each field refers to a distinct piece of data being submitted as part of the request. 

### Fields

A field consists of a unique **field name**, along with a set of attributes.  The following attributes are defined:

#### `sanitizers` (optional)

The `sanitizers` attribute specifies an ordered list of **sanitizers** to be applied to the field.

#### `validators` (optional)

The `validators` attribute specifies an ordered list of **validators** to be applied to the field.

#### `default` (optional)

The `default` attribute specifies a default value to be used if the field has not been specified in the HTTP request.  When a default value is applied, the sanitizers and validators for the field shall be ignored.

### Sanitizers

A sanitizer consists of a **sanitizer name**, and a set of sanitizer attributes.  The implementation may set one or more of these sanitizers as a default, if none are specified.

The following sanitizers are currently supported:

#### `purge`

Sanitize this field by removing all HTML entities (`'"<>&` and characters with ASCII value less than 32)

#### `escape`

Sanitize this field by escaping all HTML entities (`'"<>&` and characters with ASCII value less than 32)

#### `purify`

Sanitize this field by applying an HTML purification library, for example [HTMLPurifier](http://htmlpurifier.org/), to remove any potentially dangerous HTML code.

### Validators

A validator consists of a **validator name**, and a set of validator attributes.  The implementation may set one or more of these validators as a default, if none are specified.

The following validators are currently defined:

#### `required`

Specifies that the field is a required field.  If the field is not present in the HTTP request, the implementation shall cause this validator to fail unless a default value has been specified for the field.

#### `equals`

Specifies that the value of the field must be equivalent to a specific value.  The definition of equivalence shall be left to the implementation.

- `value` (required) : the value to which the field value must be equivalent, as defined by the implementation.

#### `not_equals`

Specifies that the value of the field must **not** be equivalent to a specific value.  The definition of equivalence shall be left to the implementation.

- `value` (required) : the value to which the field value must **not** be equivalent, as defined by the implementation.

#### `email`

Specifies that the value of the field must represent a valid email address.  The definition of a valid email address shall be left to the implementation.

#### `telephone`

Specifies that the value of the field must represent a valid telephone number.  The definition of a valid telephone number shall be left to the implementation.

#### `uri`

Specifies that the value of the field must represent a valid Uniform Resource Identifier (URI).  The definition of a valid URI shall be left to the implementation.

#### `regex`

Specifies that the value of the field must match a specified Javascript- and PCRE-compliant regular expression.

- `regex` (required): A valid Javascript- and PCRE-compliant regular expression.

#### `length`

Specifies bounds on the length, in characters, of the field's value.  The `length` validator supports the following attributes:

- `min` (optional): the minimum number of permitted characters, inclusive.  Must be a non-negative integer.
- `max` (optional): the maximum number of permitted characters, inclusive.  Must be a non-negative integer.

#### `integer`

Specifies that the value of the field must represent an integer value.

#### `numeric`

Specifies that the value of the field must represent a numeric (floating-point or integer) value.

#### `range`

Specifies a numeric interval bound on the field's value.  The `range` validator supports the following attributes:

- `min` (optional): the minimum value.  Must be a floating-point number or integer.
- `max` (optional): the maximum value.  Must be a floating-point number or integer.
- `min_exclusive` (optional): a boolean value, specifying whether the minimum value should be excluded from the interval.  The default value shall be `false`.
- `max_exclusive` (optional): a boolean value, specifying whether the maximum value should be excluded from the interval.  The default value shall be `false`.

#### `member_of`

Specifies that the value of the field must be equivalent to at least one member of a given collection.  The definition of equivalence shall be left to the implementation.

- `values` (required): A collection of values in which to search for the field value.

#### `not_member_of`

Specifies that the value of the field must not be equivalent to any members of a given collection.  The definition of equivalence shall be left to the implementation.

- `values` (required): A collection of values in which to search for the field value.

#### `matches`

Specifies that the value of the field must be equivalent to the value of another field.  The definition of equivalence shall be left to the implementation.

- `field_name` (required): The name of the other field that this field must match.  If the value of the other field is not specified in the request, this validator may attempt to match the default value, if specified.

#### `not_matches`

Specifies that the value of the field must **not** be equivalent to the value of another field.  The definition of equivalence shall be left to the implementation.

- `field_name` (required): The name of the other field that this field must **not** match.  If the value of the other field is not specified in the request, this validator may attempt to match the default value, if specified.

### Validator Messages

Additionally, each validator may contain a collection of zero or more **validation messages** assigned to a `messages` attribute.  These messages can be used by the implementation to indicate the specific point of failure during the validation process.  Each validation message shall consist of a `locale`, and a `message`.

- `locale` (required): A valid [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag).  The implementation can use this to determine which message to display to a user, based on their language and location.  A `default` locale may be specified as a fallback.
- `message` (required): A string containing the message to display to the user.

## Examples

The following is an example of a schema written in JSON format.  Notice that the names of validators, fields, etc are used as keys.  This is a possible, but not necessary, implementation of the standard.

```
{
    "user_name" : {
        "validators" : {
            "length" : {
                "min" : 1,
                "max" : 50,
                "messages" : {
                    "default" : "'Username' must be between 1 and 50 characters long.",
                    "es_US" : "Su nombre de usuario debe estar entre 1 y 50 caracteres de longitud"
                }
            },
            "required" : {
                "messages" : {
                    "default" : "'User name' is required.",
                    "es_US" : "Por favor, ingrese su nombre de usuario"
                }
            }
        },
        "sanitizers" : {
            "escape" : {}
        }
    },       
    "email" : {
        "validators" : {
            "required" : {
                "messages" : {
                    "default" : "You must specify an email address."
                }
            },
            "length" : {
                "min" : 1,
                "max" : 150,
                "messages" : {
                    "default" : "'Email' must be between 1 and 150 characters long.",
                    "es_US" : "Dirección de correo electrónico debe estar entre 1 y 150 caracteres de longitud"
                }
            },
            "email" : {
                "messages" : {
                    "default" : "'Email' must be a valid email address.",
                    "es_US" : "Dirección de correo electrónico no válida"
                }
            }
        }
    },
    "message" : {
        "default" : "My message", 
        "sanitizers" : {
            "purify" : {}
        }
    },
    "password" : {
        "validators" : {
            "required" : {
                "messages" : {
                    "default" : "'Password' is required.",
                    "es_US" : "Por favor, ingrese su contraseña"
                }
            },
            "length" : {
                "min" : 8,
                "max" : 50,
                "messages" : {
                    "default" : "'Password' must be between 8 and 50 characters long.",
                    "es_US" : "La contraseña debe tener entre 8 y 50 caracteres de longitud"
                }
            },
            "matches" : {
                "field_name" : "password_confirm",
                "messages" : {
                    "default" : "'Password' and 'Confirm password' must have the same value.",
                }
            }
        }      
    },
    "password_confirm" : {
        "validators" : {
            "matches" : {
                "field_name" : "password",
                "messages" : {
                    "default" : "'Password' and 'Confirm password' must have the same value.",
                }
            }
        }      
    }
}
```
