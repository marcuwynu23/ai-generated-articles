# Common HTTP Status Codes and When to Implement Them in API or HTTP Response

| Status Code | Meaning | When to Implement |
|-------------|---------|-------------------|
| 200 OK | The request has succeeded | Implement when the request was successful |
| 201 Created | The request has been fulfilled and a new resource has been created | Implement when a new resource has been successfully created |
| 204 No Content | The server has successfully fulfilled the request, but there is no additional content to send in the response | Implement when the request was successful, but there is no content to return |
| 400 Bad Request | The server cannot or will not process the request due to a client error | Implement when there is a problem with the request data, such as missing or invalid parameters |
| 401 Unauthorized | The client must authenticate itself to get the requested response | Implement when the user is not authenticated or the authentication token is invalid |
| 403 Forbidden | The client does not have access rights to the content, so the server is refusing to give the requested response | Implement when the user is authenticated, but does not have permission to access the requested resource |
| 404 Not Found | The server cannot find the requested resource | Implement when the requested resource does not exist |
| 500 Internal Server Error | A generic error message, given when an unexpected condition was encountered and no more specific message is suitable | Implement when there is an unexpected error on the server |
