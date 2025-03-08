# API Design/Implementation phases

1. Design RAML in Designer → Generates an API Definition in Designer 
    1. Repo for RAML is a different life cycle than API Impl Repo : https://www.mulesoft.com/resources/api/what-is-full-lifecycle-api-management
1. Execute the API Pipeline Creator (Github Actions) :
    1. Complete the required fields
    1. Create a new API Folder structure in the main repository 
    1. Use the archetype as a base
    1. Connect to Design Center to download the API definition into the new folder structure 
1. Import the API from Designer with Studio
1. Implement the API 
1. When PR is merged to master
    1. Pipeline Steps (Github Actions) :
        1. Update the API RAML in Designer 
        1. Publish the API in API Manager
            1. Create a zip file of the api folder
            1. Upload the zip file with the API info using the POST https://anypoint.mulesoft.com/apimanager/api/v1/organizations/{organizationId}/environments/{environmentId}/apis  to create the API
            1. Update in the code the apiId in the Autodiscovery property
        1. Publish the API Implementation into Runtime Manager
        1. Publish the API in Exchange
1. When the API in Design center changes:
    1. Pipeline Steps (Github Actions):
        1. Create a new branch in Github
        1. Apply changes from Design Center into the RAML copy
        1. Commit the changes in the Branch    
