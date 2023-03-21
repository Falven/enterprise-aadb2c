# Enterprise AADB2C Frontend

Fran Aguilera
<fraguile@microsoft.com>

The following document contains recommendations, including samples, for an enterprise-grade AADB2C frontend application.

---

## Branching

This is the only branch that a developer should interact with, as the main branch is protected from direct commits. Only members of the Information Security team can commit directly to the main branch.

| Branch Off | Merge/Pull Request | Branching Naming Convention |
| ---------- | ------------------ | --------------------------- |
| main       | main               | feature-\*                  |

---

### Configure CORS for IISExpress

1. Open File explorer and navigate to **C:\Program Files\IIS Express\AppServer**
2. Open the **applicationhost.config** file with an Administrative rights
3. Search for httpProtocol and you should see this:

   ```xml
   <httpProtocol>
      <customHeaders>
         <clear />
         <add name="X-Powered-By" value="ASP.NET" />
      </customHeaders>
      <redirectHeaders>
         <clear />
      </redirectHeaders>
   </httpProtocol>
   ```

4. Now add this to the customHeaders node:

   ```xml

   <add name="Access-Control-Allow-Origin" value="*" />
   <add name="Access-Control-Allow-Headers" value="Content-Type" />

   ```

5. Add this mapping under the staticContent node.

   ```xml
   <mimeMap fileExtension=".webp" mimeType="image/webp" />
   ```

Note: Original documentation at [ENABLING CORS IN IISEXPRESS](https://blog.jonathanchannon.com/2013-09-16-enabling-cors-in-iisexpress/)

---

### Access Website

Open your favorite browser and navigate to:
https://tenant.domain.com/tenant.onmicrosoft.com/oauth2/v2.0/authorize?p=Policy&client_id=GUID&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login

---

## Frontend

For more information on how these assets are developed and consumed, see [this document](https://docs.microsoft.com/en-us/azure/active-directory-b2c/customize-ui-with-html?pivots=b2c-custom-policy). The project assumes that these files will be uploaded to an Azure blob storage container.

In order to build the policies, environment variables need to be specified in a file with the name `.env` with the following form:

- `AZURE_STORAGE_ACCOUNT_URL`: Storage account where the assets are uploaded

In order to build and upload the policies:

- Make sure the `.env` file is created and up to date (see the paragraphs above).
- Run `npm install`.
- Run `npm run build`.
- A folder called `out` should now have been created; upload its contents (but not the folder itself) to the blob storage container.

---

## Image Optimization

Prefer to use [responsive image selection](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).

Compress and resize images into webp format using [cwebp](https://developers.google.com/speed/webp/docs/cwebp).

```shell
cwebp -q 80 unified-bg.png -resize 1440 900 -o unified-bg-900p.webp
```
