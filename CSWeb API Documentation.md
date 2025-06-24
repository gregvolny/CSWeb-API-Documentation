# CSPro Sync API Documentation

This document provides an overview of the CSPro Sync API endpoints, along with Javascript examples for each.

## API Endpoints

Here's a breakdown of the available API endpoints:

---
### Dictionaries

#### List Dictionaries
* **GET** `/api/dictionaries`
* **Summary:** List dictionaries
* **Description:** List names of available data dictionaries on this server
* **Javascript Example:**
```javascript
async function listDictionaries() {
  const response = await fetch('/api/dictionaries');
  const data = await response.json();
  console.log(data);
}

listDictionaries();
```

#### Add New Dictionary
* **POST** `/api/dictionaries`
* **Summary:** Add new dictionary
* **Description:** Upload a new data dictionary to the server so that data (cases) may be associated with it.
* **Parameters:**
  * `dictionary` (body, required): CSPro data dictionary as plain text
* **Javascript Example:**
```javascript
async function addDictionary(dictionaryText) {
  const response = await fetch('/api/dictionaries', {
    method: 'POST',
    headers: {
      'Content-Type': 'text/plain'
    },
    body: dictionaryText
  });
  console.log(response.status);
}

// Example usage:
// addDictionary("your dictionary text here");
```

#### Get Dictionary by Name
* **GET** `/api/dictionaries/{dictName}`
* **Summary:** Get dictionary by name
* **Description:** Download the dictionary named dictName
* **Parameters:**
  * `dictName` (path, required): Name of dictionary to download
* **Javascript Example:**
```javascript
async function getDictByName(dictName) {
  const response = await fetch(`/api/dictionaries/${dictName}`);
  const data = await response.text();
  console.log(data);
}

// Example usage:
// getDictByName("yourDictionaryName");
```

#### Delete Dictionary
* **DELETE** `/api/dictionaries/{dictName}`
* **Summary:** Delete dictionary
* **Description:** Delete a dictionary on the server and all associated data
* **Parameters:**
  * `dictName` (path, required): Name of dictionary to delete
* **Javascript Example:**
```javascript
async function deleteDictionary(dictName) {
  const response = await fetch(`/api/dictionaries/${dictName}`, {
    method: 'DELETE'
  });
  console.log(response.status);
}

// Example usage:
// deleteDictionary("yourDictionaryName");
```

---
### Cases

#### Get Cases for Dictionary
* **GET** `/api/dictionaries/{dictName}/cases`
* **Summary:** Get cases for dictionary
* **Description:** Download cases for the dictionary named dictName
* **Parameters:**
  * `dictName` (path, required): Name of dictionary to get cases from
  * `x-csw-universe` (header): universe to limit cases returned
  * `x-csw-case-range-start-after` (header): only return cases whose guid is strictly greater (alphabetically) than this value. Used for paging results.
  * `x-csw-case-range-count` (header): limit number cases returned to this number of cases.
  * `x-csw-exclude-revisions` (header): exclude cases from following revisions from the result. Allows device to avoid downloading cases that it previously uploaded.
  * `x-csw-device` (header): id of device requesting cases
  * `If-Match` (header): etag from previous call. Only cases added/modified since this etag are returned. If server does not have this etag it will return status 412.
* **Javascript Example:**
```javascript
async function getCasesForDict(dictName, options = {}) {
  const headers = {};
  if (options.universe) headers['x-csw-universe'] = options.universe;
  if (options.startAfter) headers['x-csw-case-range-start-after'] = options.startAfter;
  if (options.count) headers['x-csw-case-range-count'] = options.count;
  if (options.excludeRevisions) headers['x-csw-exclude-revisions'] = options.excludeRevisions.join(',');
  if (options.device) headers['x-csw-device'] = options.device;
  if (options.ifMatch) headers['If-Match'] = options.ifMatch;

  const response = await fetch(`/api/dictionaries/${dictName}/cases`, { headers });
  const data = await response.json();
  console.log(data);
}

// Example usage:
// getCasesForDict("yourDictionaryName", { count: 10 });
```

#### Add or Update New Cases to Dictionary
* **POST** `/api/dictionaries/{dictName}/cases`
* **Summary:** Add or update new cases to dictionary
* **Description:** Upload new cases to the dictionary named dictName
* **Parameters:**
  * `dictName` (path, required): Name of dictionary to add case to
  * `case` (body, required): Cases to add/update
  * `x-csw-device` (header, required): id of device sending cases
  * `If-Match` (header): etag from previous call. If server does not have this etag it will return status 412.
* **Javascript Example:**
```javascript
async function addOrUpdateCase(dictName, caseData, deviceId, ifMatch) {
  const headers = {
    'Content-Type': 'application/json',
    'x-csw-device': deviceId
  };
  if (ifMatch) headers['If-Match'] = ifMatch;

  const response = await fetch(`/api/dictionaries/${dictName}/cases`, {
    method: 'POST',
    headers,
    body: JSON.stringify(caseData)
  });
  console.log(response.status);
}

// Example usage:
// const caseList = [ /* your case objects */ ];
// addOrUpdateCase("yourDictionaryName", caseList, "yourDeviceId");
```

#### Get Case by ID
* **GET** `/api/dictionaries/{dictName}/cases/{caseId}`
* **Summary:** Get case by id
* **Description:** Download case for the dictionary named dictName with id caseId
* **Parameters:**
  * `dictName` (path, required): Name of dictionary to get cases from
  * `caseId` (path, required): Id of case to download (GUID)
* **Javascript Example:**
```javascript
async function getCaseById(dictName, caseId) {
  const response = await fetch(`/api/dictionaries/${dictName}/cases/${caseId}`);
  const data = await response.json(); // Or response.text() depending on Accept header
  console.log(data);
}

// Example usage:
// getCaseById("yourDictionaryName", "yourCaseId");
```

#### Update Existing Case
* **PUT** `/api/dictionaries/{dictName}/cases/{caseId}`
* **Summary:** Update existing case
* **Description:** Overwrite existing case for the dictionary named dictName with id caseId
* **Parameters:**
  * `dictName` (path, required): Name of dictionary containing case
  * `caseId` (path, required): Id of case to update (GUID)
  * `case` (body, required): Case to update
* **Javascript Example:**
```javascript
async function updateCase(dictName, caseId, caseData) {
  const response = await fetch(`/api/dictionaries/${dictName}/cases/${caseId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json' // Or 'text/plain'
    },
    body: JSON.stringify(caseData) // Or text representation
  });
  console.log(response.status);
}

// Example usage:
// const caseObject = { /* your case object */ };
// updateCase("yourDictionaryName", "yourCaseId", caseObject);
```

#### Delete Existing Case
* **DELETE** `/api/dictionaries/{dictName}/cases/{caseId}`
* **Summary:** Delete existing case
* **Description:** Delete existing case from the dictionary named dictName with id caseId
* **Parameters:**
  * `dictName` (path, required): Name of dictionary containing case
  * `caseId` (path, required): Id of case to delete (GUID)
* **Javascript Example:**
```javascript
async function deleteCase(dictName, caseId) {
  const response = await fetch(`/api/dictionaries/${dictName}/cases/${caseId}`, {
    method: 'DELETE'
  });
  console.log(response.status);
}

// Example usage:
// deleteCase("yourDictionaryName", "yourCaseId");
```

---
### Sync History

#### Get Sync History
* **GET** `/api/dictionaries/{dictName}/syncs`
* **Summary:** Get sync history
* **Description:** List synchronizations for the server
* **Parameters:**
  * `dictName` (path, required): Name of dictionary containing case
  * `from` (query): Start date in RFC3339 format
  * `to` (query): End date in RFC3339 format
  * `deviceId` (query): Device Id
  * `limit` (query): maximum number of cases in response
  * `offset` (query): zero-based index of first case to return in response
* **Javascript Example:**
```javascript
async function getSyncHistory(dictName, params = {}) {
  const queryParams = new URLSearchParams(params);
  const response = await fetch(`/api/dictionaries/${dictName}/syncs?${queryParams}`);
  const data = await response.json();
  console.log(data);
}

// Example usage:
// getSyncHistory("yourDictionaryName", { limit: 20 });
```

---
### Files

#### Get File Info
* **GET** `/api/files/{filePath}`
* **Summary:** Get file info
* **Description:** Get contents of file matching path.
* **Parameters:**
  * `filePath` (path, required): path of file to download for
* **Javascript Example:**
```javascript
async function getFileInfo(filePath) {
  const response = await fetch(`/api/files/${filePath}`);
  const data = await response.json();
  console.log(data);
}

// Example usage:
// getFileInfo("path/to/your/file.txt");
```

#### Get File Content
* **GET** `/api/files/{filePath}/content`
* **Summary:** Get file content
* **Description:** Get contents of file matching path.
* **Parameters:**
  * `filePath` (path, required): path of file to download
  * `If-None-Match` (header): ETag value from previous response
* **Javascript Example:**
```javascript
async function getFileContent(filePath, ifNoneMatch) {
  const headers = {};
  if (ifNoneMatch) headers['If-None-Match'] = ifNoneMatch;

  const response = await fetch(`/api/files/${filePath}/content`, { headers });
  // Handle response based on content type (e.g., response.blob(), response.text())
  console.log(response.status, response.headers.get('ETag'));
}

// Example usage:
// getFileContent("path/to/your/file.txt");
```

#### Upload File Content
* **PUT** `/api/files/{filePath}/content`
* **Summary:** Upload file
* **Description:** Put contents of file to server directory specified by filePath. Create directory filePath if it does not exist.
* **Parameters:**
  * `filePath` (path, required): Full path to upload file to
  * `Content-MD5` (header, required): Signature of file contents
* **Javascript Example:**
```javascript
async function putFileContent(filePath, fileContent, contentMd5) {
  const response = await fetch(`/api/files/${filePath}/content`, {
    method: 'PUT',
    headers: {
      'Content-MD5': contentMd5,
      // 'Content-Type': 'application/octet-stream' // Or appropriate content type
    },
    body: fileContent // Should be a Blob, BufferSource, FormData, URLSearchParams, or ReadableStream
  });
  const data = await response.json();
  console.log(data);
}

// Example usage:
// const myFile = new File(["file content"], "example.txt", { type: "text/plain" });
// const md5Hash = "your_md5_hash_of_file_content"; // Calculate MD5 hash
// putFileContent("path/to/upload/example.txt", myFile, md5Hash);
```

---
### Folders

#### Get Files in Folder
* **GET** `/api/folders/{folderPath}`
* **Summary:** Get files in folder
* **Description:** Get file info for files in the given folder
* **Parameters:**
  * `folderPath` (path, required): path of folder to list contents of
* **Javascript Example:**
```javascript
async function getFilesInFolder(folderPath) {
  const response = await fetch(`/api/folders/${folderPath}`);
  const data = await response.json();
  console.log(data);
}

// Example usage:
// getFilesInFolder("path/to/your/folder");
```

---
### Server

#### Get Server Details
* **GET** `/api/server`
* **Summary:** Get server details
* **Description:** Get information about server
* **Javascript Example:**
```javascript
async function getServerInfo() {
  const response = await fetch('/api/server');
  const data = await response.json();
  console.log(data);
}

getServerInfo();
```

---
### Users

#### Get List of Users
* **GET** `/api/users`
* **Summary:** Get list of users
* **Javascript Example:**
```javascript
async function getUsers() {
  const response = await fetch('/api/users');
  const data = await response.json();
  console.log(data);
}

getUsers();
```

#### Get User by Username
* **GET** `/api/users/{username}`
* **Summary:** Get user by user name
* **Parameters:**
  * `username` (path, required): The name that needs to be fetched.
* **Javascript Example:**
```javascript
async function getUserByName(username) {
  const response = await fetch(`/api/users/${username}`);
  const data = await response.json();
  console.log(data);
}

// Example usage:
// getUserByName("aUser");
```

#### Update User
* **PUT** `/api/users/{username}`
* **Summary:** Updated user
* **Description:** This can only be done by the logged in user.
* **Parameters:**
  * `username` (path, required): name of user to update
  * `body` (body): Updated user object
* **Javascript Example:**
```javascript
async function updateUser(username, userData) {
  const response = await fetch(`/api/users/${username}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(userData)
  });
  console.log(response.status);
}

// Example usage:
// const updatedUserInfo = { firstName: "NewName" };
// updateUser("aUser", updatedUserInfo);
```

#### Delete User
* **DELETE** `/api/users/{username}`
* **Summary:** Delete user
* **Description:** This can only be done by the logged in user.
* **Parameters:**
  * `username` (path, required): The name that needs to be deleted
* **Javascript Example:**
```javascript
async function deleteUser(username) {
  const response = await fetch(`/api/users/${username}`, {
    method: 'DELETE'
  });
  console.log(response.status);
}

// Example usage:
// deleteUser("aUser");
```

---
### Apps

#### Get List of Applications
* **GET** `/api/apps`
* **Summary:** Get list of applications
* **Javascript Example:**
```javascript
async function getApps() {
  const response = await fetch('/api/apps');
  const data = await response.json();
  console.log(data);
}

getApps();
```

#### Get App Package
* **GET** `/api/apps/{appname}`
* **Summary:** Get app package
* **Description:** Get application deployment package (csapb file) for named application
* **Parameters:**
  * `appname` (path, required): name of app to download
  * `If-None-Match` (header): Signature (MD5) of current application package specification on client.
  * `x-csw-package-build-time` (header): Build date/time of application package on client.
  * `x-csw-package-files` (header): List of application package files and signatures.
* **Javascript Example:**
```javascript
async function getApp(appname, options = {}) {
  const headers = {};
  if (options.ifNoneMatch) headers['If-None-Match'] = options.ifNoneMatch;
  if (options.packageBuildTime) headers['x-csw-package-build-time'] = options.packageBuildTime;
  if (options.packageFiles) headers['x-csw-package-files'] = options.packageFiles;

  const response = await fetch(`/api/apps/${appname}`, { headers });
  // Handle file download (e.g., response.blob())
  console.log(response.status, response.headers.get('Content-Length'));
}

// Example usage:
// getApp("yourAppName");
```

#### Upload App Package
* **PUT** `/api/apps/{appname}`
* **Summary:** Upload app package
* **Description:** Upload application package (csapb file) to server.
* **Parameters:**
  * `appname` (path, required): Name of app to upload
* **Javascript Example:**
```javascript
async function updateApp(appname, packageFile) {
  const response = await fetch(`/api/apps/${appname}`, {
    method: 'PUT',
    // headers: { 'Content-Type': 'application/octet-stream' }, // Set appropriate content type
    body: packageFile // Should be a Blob, BufferSource, FormData, URLSearchParams, or ReadableStream representing the .csapb file
  });
  console.log(response.status);
}

// Example usage:
// const myCsapbFile = new File(["...csapb content..."], "myApp.csapb");
// updateApp("yourAppName", myCsapbFile);
```

#### Delete App
* **DELETE** `/api/apps/{appname}`
* **Summary:** Delete app
* **Parameters:**
  * `appname` (path, required): The name of the app to be deleted
* **Javascript Example:**
```javascript
async function deleteApp(appname) {
  const response = await fetch(`/api/apps/${appname}`, {
    method: 'DELETE'
  });
  console.log(response.status);
}

// Example usage:
// deleteApp("yourAppName");
```
