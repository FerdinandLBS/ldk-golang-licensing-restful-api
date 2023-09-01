## Documentation for API Endpoints

All URIs are relative to *https://localhost:8088/sentinel/ldk_runtime/v1*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*LicenseApi* | [**GetFeatureInfo**](docs/LicenseApi.md#getfeatureinfo) | **Get** /vendors/{vendorId}/features | getFeatureInfo
*LicenseApi* | [**GetKeyInfo**](docs/LicenseApi.md#getkeyinfo) | **Get** /vendors/{vendorId}/keys | getKeyInfo
*LicenseApi* | [**GetProductInfo**](docs/LicenseApi.md#getproductinfo) | **Get** /vendors/{vendorId}/products | getProductInfo
*LicenseApi* | [**Login**](docs/LicenseApi.md#login) | **Post** /vendors/{vendorId}/sessions | login
*LicenseApi* | [**Logout**](docs/LicenseApi.md#logout) | **Delete** /vendors/{vendorId}/sessions/{sessionId} | logout
*LicenseApi* | [**Refresh**](docs/LicenseApi.md#refresh) | **Post** /vendors/{vendorId}/sessions/{sessionId}/refresh | refresh


## Documentation For Models

 - [ClientInfo](docs/ClientInfo.md)
 - [Concurrency](docs/Concurrency.md)
 - [ErrorInfo](docs/ErrorInfo.md)
 - [FeatureInfo](docs/FeatureInfo.md)
 - [KeyInfo](docs/KeyInfo.md)
 - [LicenseInfo](docs/LicenseInfo.md)
 - [LicenseRequest](docs/LicenseRequest.md)
 - [LicenseResponse](docs/LicenseResponse.md)
 - [ProductInfo](docs/ProductInfo.md)
 - [Scope](docs/Scope.md)
 - [SessionInfo](docs/SessionInfo.md)


## Documentation For Authorization
 Endpoints do not require authorization.


## Documentation For Sample


	// parse & validate environment variables
	godotenv.Load()
	flags.Parse(&env)

	// parse the client identity
	clientIdResult := strings.Split(env.ClientIdentity, ":")
	if clientIdResult == nil || len(clientIdResult) != 2 {
		log.Fatal("Client Identity is not valid")
		return
	}

	authCtx := context.WithValue(context.Background(), api.ContextIdentity, api.IdentityAuth{
		Id:     clientIdResult[0],
		Secret: clientIdResult[1],
	})

	cfg := &api.Configuration{
		Host:     env.ServerAddr,
		VendorId: env.VendorId,
		Scheme:   env.EndpointScheme,
		BasePath: env.EndpointScheme + "://" + env.ServerAddr + ":" + env.ServerPort + "/sentinel/ldk_runtime/v1",
	}

	licensingApiClient := api.NewAPIClient(cfg)
	licenseRequest := api.LicenseRequest{}
	licenseRequest.FeatureId = 0

	// get client info
	user, err := user.Current()
	if err != nil {
		log.Fatalf(err.Error())
	}

	licenseRequest.ClientInfo = &api.ClientInfo{}
	licenseRequest.ClientInfo.MachineId, _ = machineid.ID()
	licenseRequest.ClientInfo.UserName = user.Username
	licenseRequest.ClientInfo.DomainName, _ = os.Hostname()
	licenseRequest.ClientInfo.ProcessId = strconv.Itoa(os.Getpid())
	licenseRequest.ClientInfo.ClientDateTime = time.Now().UTC().Format(time.RFC3339)

	apiResponse, _, err := licensingApiClient.LicenseApi.Login(authCtx, licenseRequest)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Printf("licensingApi.LicenseApi.Login %#v", apiResponse)

	localVarOptionals := &api.QueryInfoOpts{
		PageStartIndex: optional.NewInt32(0),
		PageSize:       optional.NewInt32(1),
	}
	keys, _, err := licensingApiClient.LicenseApi.GetKeyInfo(authCtx, localVarOptionals)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Printf("licensingApi.LicenseApi.GetKeyInfo %#v", keys)

	products, _, err := licensingApiClient.LicenseApi.GetProductInfo(authCtx, localVarOptionals)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Printf("licensingApi.LicenseApi.GetProductInfo %#v", products)

	features, _, err := licensingApiClient.LicenseApi.GetFeatureInfo(authCtx, localVarOptionals)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Printf("licensingApi.LicenseApi.GetFeatureInfo %#v", features)

	licenseResponse, _, err := licensingApiClient.LicenseApi.Refresh(authCtx, apiResponse.SessionId)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Printf("licensingApi.LicenseApi.Refresh  %#v", licenseResponse)

	_, err = licensingApiClient.LicenseApi.Logout(authCtx, apiResponse.SessionId)
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Println("licensingApi.LicenseApi.Logout", apiResponse.SessionId)

