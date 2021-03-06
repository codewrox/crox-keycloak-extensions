- [Client Auth Validation Keycloak Extension](#client-auth-validation-keycloak-extension)
- [Understanding the template](#understanding-the-template)
- [Available functions for validation](#available-functions-for-validation)
  - [Using attributes](#using-attributes)
    - [user.getAttributeValue](#usergetattributevalue)
    - [user.hasAttribute](#userhasattribute)
  - [Using groups](#using-groups)
    - [user.hasGroup](#userhasgroup)
  - [Using roles](#using-roles)
    - [user.hasRole](#userhasrole)
  - [Using realm client parameters](#using-realm-client-parameters)
    - [user.getClientId](#usergetclientid)
    - [user.getClientName](#usergetclientname)
    - [user.getClientBaseUrl](#usergetclientbaseurl)
    - [user.getClientRootUrl](#usergetclientrooturl)
  - [Custom Calc Values](#custom-calc-values)
    - [user.set](#userset)
- [Redirect Url Template](#redirect-url-template)
- [Installation](#installation)
  - [Configuration steps](#configuration-steps)
    - [Step-1: Installation of "Codewrox - Cookie Client Auth Validation"](#step-1-installation-of-codewrox---cookie-client-auth-validation)
    - [Step-2 Installation of "Codewrox - Client Auth Validation"](#step-2-installation-of-codewrox---client-auth-validation)
    - [Step-3 Configuring the extenstions](#step-3-configuring-the-extenstions)
# Client Auth Validation Keycloak Extension

This extension provides you more granular control over the login flow for both SAML and OpenID clients.  Using this extension, one can restrict access to a particular client or set of clients based on the following validation criteria 

- User attributes 
- Group attributes
- Roles
- Groups
- Realm Client parameters

You can use any one of the criteria or in combination of multiple criteria to validate an user is allowed to login or redirect to a different location if not allowed. So setting up a criteria is very much configurable as a template. This extension offers you a custom redirect url locaton where in you can configure to redirect if the validation criteria is not met.


# Understanding the template

The template is configured after your setup your execution flow as described in the installation step below but let's understand the usage of it before then.

The template uses a keyword called "user" that is basically representing an user currently attempting to login. Technically to say, "user" is a Java object by itself that is exposed into this template for you to describe your various conditions.  So how do i use this "user" keyword in my template? Let's understand from a simple example that suppose, you want to restrict an user for a particular realm client who do not have an EmployeeID set in the user attribute.


```
 

<#if ( user.getAttributeValue("EmployeeID") != "" )  >

    ${ user.set("allow_access", true)  } 

</#if>


```


> "allow_access" is an important value to set to determine the validation pass or fail at the end


Here is the complex one where you can use logical operators like and/or/not conditions and nested "if" conditions as like below

```

<#if ( ( user.getAttributeValue("EmployeeID") != "" ) && user.hasGroup("HR-Admins") )  >

    ${ user.set("allow_access", true)  } 

<#else>

  <#if ( user.hasRole("Administrators") || user.hasGroup("Keycloak-Admins") )  >

    ${ user.set("allow_access", true)  } 

   </#if>

</#if>


```


# Available functions for validation

You can use the following functions in your template for designing your  validation criteria.

##  Using attributes

### user.getAttributeValue

```
Syntax:  user.getAttributeValue("<attribute_name>")

It returns empty string if the attribute is not defined or null

Note: if the attribute value is not found for the user then it fall back on to group level attributes.  

```


### user.hasAttribute

```
Syntax:  user.hasAttribute("<attribute_name>")

It returns false if the attribute name is not defined.  It's good for checking an attribute is defined or not in the system.   

Note: if the attribute name is not found for the user then it fall back on to group level attributes to continue verify.  

```


##  Using groups

### user.hasGroup


```
Syntax:  user.hasGroup("<group_name>")
where, group_name can be just a group name or it can be specified with the parent path for example, HR-Groups/Hiring-Managers

It returns false if the user who is attempting to login does not belong to the group.  



```

##  Using roles

### user.hasRole


```
Syntax:  user.hasGroup("<role_name>")

It returns false if the user who is attempting to login has not been assigned to the role.  
 
Note: if the role is not found for the user then it fall back on to group level roles to continue verify.  
```


##  Using realm client parameters

### user.getClientId

```
Syntax:  user.getClientId()

It returns ID value of the client that the user is attempting to login.
 
```

### user.getClientName

```
Syntax:  user.getClientName()

It returns Name of the client that the user is attempting to login.
 
```


### user.getClientBaseUrl

```
Syntax:  user.getClientBaseUrl()

It returns base url of the client that the user is attempting to login.
 
```


### user.getClientRootUrl

```
Syntax:  user.getClientRootUrl()

It returns root url of the client that the user is attempting to login.
 
```

##  Custom Calc Values

In your template, you can set any calculated values of your choice using two utility functions called get() and set() as explained below. This can be exteremely useful for custom redirect_url formation which is explained in next topic.  For example, if an user belongs to a country then you may want to append a country code as part your redirect url dynamically per user per client basis.

### user.set

```
Syntax:  user.set("<variable_name>", <custom_value>)
where, variable_name is your identifier where the value stored into
       custom_value can be a boolean or integer or a string value(needs double quote)

It's more like a key value pair. 
 
```

# Redirect Url Template

This exension has an option to specify where to redirect if the specified validation criteria is not satisfied.  Please note that this is also a template where in you can use all the functions mentioned above.  For example,

```
Redirect url :  https://codewrox.com/${ user.get("country_code")}?employeeId=${ user.getAttributeValue("EmployeeID")}

```
 
# Installation

Simply copy the JAR into `/opt/jboss/keycloak/standalone/deployments/`

## Configuration steps

There are two scenarios to be validated 

- Step-1) First time login which means browser does not have access tokens but it needs to be validated. 
- Step-2) Already logged in with a client and has a valid access tokens available but other client needs to be validated before SSO triggers (SAML/OpenID)
- Step-3) Finally, configuring the extenstions

### Step-1: Installation of "Codewrox - Cookie Client Auth Validation"
- Clone the `Browser` authentication for example, "my-browser-flow"
- Add a new execution, select "Codewrox - Cookie Client Auth Validation" first from the drop down
- Important: Move this execution at the very top first row to have the first priorty and set to "Alternative" auth type
- Disable the "Cookie" execution as you may see this in second row 
- Now its the time to configure your newly added execution 
    - 

### Step-2 Installation of "Codewrox - Client Auth Validation"
- Continue on the same setup with that example "my-browser-flow"
- Locate  "My-browser-flow Forms" and select actions to add a new execution but this time you select "Codewrox - Client Auth Validation"  (do not select the one in step#1)
- Important: Leave this execution at the very bottom under the forms to have the last priorty and set to "Required" auth type
- That's all. 
  
### Step-3 Configuring the extenstions
- With the newly added two execution flow, start configuring it with the template described above.
- Mostly you would use the same criteria in both execution flow. Though, its a bit of duplicate here but that's how exetensions are designed in Keycloak and also our requirement need to tackle two different scenarios too as mentioned in the above steps.
- And finally,  do not forget to set this new authentication flow in your client. (example: "my-browser-flow")
- You can use one authentication flow for many different clients but if each client needs different validation criteria then you will have to create different authentication flow.
  


--------------------------------------------------------
Hire us for SSO integrations,  Keycloak deployments 
and customizations : info(at)codewrox.com
