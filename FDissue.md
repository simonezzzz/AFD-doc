reference to issue: https://github.com/MicrosoftDocs/azure-docs/issues/65432#issue-734956246 


503 response from Front Door
Symptom
Backend health percentage showed probe down on the backend

Cause
	1. The backend hostname is IP address while the Certificate subject name validation has enabled
	2. Misconfiguration of health probe of backend pool
	3. backend pool Misconfiguration


Troubleshooting steps
	
	1. Send the request to your backend directly (without going through Front Door) using the hostname and host header filled in the backend pool.
	
	Send the request to your Front Door.
	
	If going through the Front Door results in 503 error response code, but you can access backend pool directly:
	
	If the backend hostname is IP address, and  you can see the certificate error  when you access the backend directly:
		
		○ **Certificate subject name mismatch**: For HTTPS connections, Front Door expects that your backend presents certificate from a valid CA with subject name(s) matching the backend hostname. As an example, if your backend hostname is set to myapp-centralus.contosonews.net and the certificate that your backend presents during the TLS handshake neither has myapp-centralus.contosonews.net nor *myapp-centralus*.contosonews.net in the subject name, the Front Door will refuse the connection and result in an error.
		
		**Solution**: While it is not recommended from a compliance standpoint, you can workaround this error by disabling certificate subject name check for your Front Door. This is present under Settings in Azure portal and under - -BackendPoolsSettings in the API.
		
		○ **Backend hosting certificate from invalid CA**: Only certificates from [valid CAs](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-troubleshoot-allowed-ca) can be used at the backend with Front Door. Certificates from internal CAs or self-signed certificates are not allowed.
		
	2. Send the request to your backend directly (without going through Front Door) using the path and http method of the helth probe.
	If your backend does not accept the path and/or HTTP method configured in the custom health probe, for example, if the health probe path is "/health"  and HTTP method is POST, but there is no path called "/health" or this path does not allow POST method, the health check would fail.
	**Solution**: You can disable custom health probe, or you can specify the path and HTTP method that is available for the backend pool.
	
	3. Send the request to your backend directly (without going through Front Door) using the port, hostname and host header you have configured in the backendpool.
	If you are unable to access it, the configuration of backend pool is not appropriate.
	**Solution**:
		○ You can change the port listening on the backend host, check firewall settings did not block the traffic from IP range from service tag ["AzureFrontDoor.Backend"](https://www.microsoft.com/en-us/download/details.aspx?id=56519).
		○ You can pick a valid hostname/ host header on the backend host and fill it in the backend pool settings.
