---
title: Download a blob with JavaScript
titleSuffix: Azure Storage
description: Learn how to download a blob in Azure Storage by using the JavaScript client library.
services: storage
author: pauljewellmsft

ms.author: pauljewell
ms.date: 11/30/2022
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.devlang: javascript
ms.custom: devx-track-js, devguide-js
---

# Download a blob with JavaScript

This article shows how to download a blob using the [Azure Storage client library for JavaScript](https://www.npmjs.com/package/@azure/storage-blob). You can download a blob by using any of the following methods:

- [BlobClient.download](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-download)
- [BlobClient.downloadToBuffer](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-downloadtobuffer-1) (only available in Node.js runtime)
- [BlobClient.downloadToFile](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-downloadtofile) (only available in Node.js runtime)

> [!NOTE]
> The examples in this article assume that you've created a [BlobServiceClient](/javascript/api/@azure/storage-blob/blobserviceclient) object by using the guidance in the [Get started with Azure Blob Storage and JavaScript](storage-blob-javascript-get-started.md) article. Blobs in Azure Storage are organized into containers. Before you can upload a blob, you must first create a container. To learn how to create a container, see [Create a container in Azure Storage with JavaScript](storage-blob-container-create.md). 
 
## Download to a file path

The following example downloads a blob by using a file path with the [BlobClient.downloadToFile](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-downloadtofile) method. This method is only available in the Node.js runtime:

```javascript
async function downloadBlobToFile(containerClient, blobName, fileNameWithPath) {

    const blobClient = await containerClient.getBlobClient(blobName);
    
    await blobClient.downloadToFile(fileNameWithPath);
    console.log(`download of ${blobName} success`);
}
```

## Download as a stream

The following example downloads a blob by creating a Node.js writable stream object and then piping to that stream with the [BlobClient.download](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-download) method.

```javascript
async function downloadBlobAsStream(containerClient, blobName, writableStream) {

    const blobClient = await containerClient.getBlobClient(blobName);

    const downloadResponse = await blobClient.download();

    downloadResponse.readableStreamBody.pipe(writableStream);
    console.log(`download of ${blobName} succeeded`);
}
```

## Download to a string

The following Node.js example downloads a blob to a string with [BlobClient.download](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-download) method. In Node.js, blob data returns in a `readableStreamBody`.

```javascript

async function downloadBlobToString(containerClient, blobName) {

    const blobClient = await containerClient.getBlobClient(blobName);

    const downloadResponse = await blobClient.download();

    const downloaded = await streamToBuffer(downloadResponse.readableStreamBody);
    console.log('Downloaded blob content:', downloaded.toString());
}

async function streamToBuffer(readableStream) {
    return new Promise((resolve, reject) => {
        const chunks = [];
        readableStream.on('data', (data) => {
            chunks.push(data instanceof Buffer ? data : Buffer.from(data));
        });
        readableStream.on('end', () => {
            resolve(Buffer.concat(chunks));
        });
        readableStream.on('error', reject);
    });
}
```

If you're working with JavaScript in the browser, blob data returns in a promise [blobBody](/javascript/api/@azure/storage-blob/blobdownloadresponseparsed#@azure-storage-blob-blobdownloadresponseparsed-blobbody). To learn more, see the example usage for browsers at [BlobClient.download](/javascript/api/@azure/storage-blob/blobclient#@azure-storage-blob-blobclient-download).

## Resources

To learn more about how to download blobs using the Azure Blob Storage client library for JavaScript, see the following resources.

### REST API operations

The Azure SDK for JavaScript contains libraries that build on top of the Azure REST API, allowing you to interact with REST API operations through familiar JavaScript paradigms. The client library methods for downloading blobs use the following REST API operation:

- [Get Blob](/rest/api/storageservices/get-blob) (REST API)

### Code samples

View code samples from this article (GitHub):
- [Download to file](https://github.com/Azure-Samples/AzureStorageSnippets/blob/master/blobs/howto/JavaScript/NodeJS-v12/dev-guide/download-blob-to-file.js)
- [Download to stream](https://github.com/Azure-Samples/AzureStorageSnippets/blob/master/blobs/howto/JavaScript/NodeJS-v12/dev-guide/download-blob-to-stream.js)
- [Download to string](https://github.com/Azure-Samples/AzureStorageSnippets/blob/master/blobs/howto/JavaScript/NodeJS-v12/dev-guide/download-blob-to-string.js)

[!INCLUDE [storage-dev-guide-resources-javascript](../../../includes/storage-dev-guides/storage-dev-guide-resources-javascript.md)]
