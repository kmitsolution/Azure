
To add a certificate to the Azure Application Gateway and configure it to serve HTTPS traffic using a self-signed certificate, follow these steps:

1. **Create a Self-Signed Certificate**:
   - Open PowerShell with administrator privileges.
   - Run the following command to create a self-signed certificate:

   ```powershell
   New-SelfSignedCertificate -DnsName "kmitedu.com" -CertStoreLocation "cert:\currentuser\my"
   ```

2. **Export Certificate with Private Key**:
   - Open the Certificate Manager (certmgr.msc).
   - Navigate to the "Personal" folder and find the certificate with the subject "kmitedu.com".
   - Right-click on the certificate, select "All Tasks", then "Export".
   - Follow the export wizard, select "Yes, export the private key", and choose the PFX format. Set a password for the exported file.

3. **Upload Certificate to Azure Application Gateway**:
   - Navigate to your Azure Application Gateway resource in the Azure portal.
   - Under "Listeners", click on "Add".
   - Configure the listener with the appropriate settings, including port 443.
   - Select "Upload a certificate" and upload the PFX file exported earlier. Provide the name of the certificate and the password used during export.


4. **Create HTTPS Rule**:
   - Navigate to "Rules" under the Application Gateway resource.
   - Click on "Add a rule".
   - Configure the rule to use the HTTPS listener created earlier.
   - Select the appropriate backend pool containing your VM machines.

5. **Verify Configuration**:
   - Wait for the changes to propagate, and then access "https://kmitedu.com" in a web browser.
   - You may need to configure DNS to point to the Application Gateway's public IP if you haven't already done so.
   - Since this is a self-signed certificate, the browser may display a warning about the certificate being untrusted. Proceed to the website anyway.

After completing these steps, your Azure Application Gateway should be configured to serve HTTPS traffic using the self-signed certificate for the domain "kmitedu.com".
