# Named Credentials in Salesforce

## What are Named Credentials?
Named Credentials are a secure way of storing API callout endpoints, authentication, and access details within Salesforce. They act as a centralized point for handling external API integrations, enabling Salesforce developers to decouple endpoint details from the code. This approach not only simplifies code management but also enhances security by abstracting sensitive details from the codebase.

Before named credentials, the endpoint would have been coded into the apex class and added to Remote Site Settings to enable callouts to that api endpoint. With named credentials, this step can be skipped. Named credentials can be thought of as a variable or container for the callout endpoint. This makes the code cleaner as you will see in the example below.

## Creating a Legacy Named Credential
The steps are as follows
1. Create a Connected App
2. Create an Auth. Provider
3. Create a Named Credential

### STEP 1 — Creating a Connected App
Navigate to **Setup** -> **Apps** -> **App Manager** and click on **New Connected App**.
1. Provide a name and an API name for the connected app.
2. Provide a contact email.
3. Check the Enable OAuth Settings option under API(Enable OAuth Settings).
4. Provide a dummy URL for the Callback URL at this stage. This will be updated after the next step.
5. Select the relevant OAuth Scopes.
6. Save

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/new%20connected%20app.webp">

### STEP 2 — Create an Auth. Provider
Navigate to **Setup** -> **Identity** -> **Auth. Providers** and click on **New**.

Select a **Provider type**. In my case, I am choosing **Salesforce** to allow me to call Salesforce Rest APIs without using the current sessionID.

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/new%20auth%20provider.webp">

Complete the setup by filling in the form below

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/auth%20provider%20details.webp">

Provide a Name and URL suffix.
Note that the Customer Key and Customer Secret are created in the previous step and are part of the Connected App.

Click on Manage Consumer Details as shown below and this will prompt the user to enter a verification code sent to their email address.

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/consumer%20key%20details.webp">

The customer key and secret can be copied over to the Auth. Provider. Accept the default for the remaining fields and click Save.
This step generates several URLs including the Callback URL.

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/callback%20url.webp">

Update the **Callback URL** in the **Connected App** with this value.

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/update%20callback%20url.webp">

### STEP 3 — Create the Named Credential
Navigate to **Setup** -> **Security** -> **Named Credentials** and click on the dropdown next to New and select **New Legacy**.

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/new%20legacy.webp">

Populate the fields as shown below.
Note that the **URL** is the **My Domain** url (navigate to **My Domain** and copy the My Current Domain URL).

<img src="https://github.com/MekanJuma/Named-Credentials-in-Salesforce/blob/main/screenshots/named%20credentials.webp">

For the **Scope**, I entered **refresh_token** **full**. This ensures that the process returns a **refresh_token**, otherwise the access will expire.
On **Save**, the authentication process will kick off and redirect the user to a login page. Note that the Authentication Status will be updated after this authentication process.

## Example Usage
```apex
public class ToolingApiUtility {
    private static final String ENDPOINT_BASE = 'callout:Named_Cred_Name/services/data/v58.0/tooling/';

    public static HttpResponse makeRequest(String path, String method, String body) {
        HttpRequest request = new HttpRequest();
        request.setEndpoint(ENDPOINT_BASE + path);
        request.setMethod(method);
        request.setHeader('Content-Type', 'application/json');

        if (String.isNotBlank(body)) {
            request.setBody(body);
        }

        Http http = new Http();
        try {
            return http.send(request);
        } catch (Exception e) {
            System.debug('Error in makeRequest: ' + e.getMessage());
            throw e;
        }
    }
	
    @AuraEnabled
    public static String queryToolingAPI(String soql) {
        String path = 'query/?q=' + EncodingUtil.urlEncode(soql, 'UTF-8');
        HttpResponse response = makeRequest(path, 'GET', null);
        if (response.getStatusCode() == 200) {
            return response.getBody();
        } else {
            System.debug('Error in queryToolingAPI: ' + response.getBody());
            return null;
        }
    }
}
```

**Author**: Mekan Jumayev

