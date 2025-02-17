---
title: VPC Lattice
---

:::caution

This page is a work in progress. If you want to help us to make this page better, please consider contributing on GitHub.

:::

We recommend using `@middy/http-event-normalizer` if you place to use any of the following: `@middy/http-json-body-parser`, `@middy/http-multipart-body-parser`, `@middy/http-urlencode-body-parser`, `@middy/http-partial-response`

## AWS Documentation

- [Using AWS Lambda with Amazon VPC Lattice](https://docs.aws.amazon.com/lambda/latest/dg/services-vpc-lattice.html)

## Example

```javascript
import middy from '@middy/core'
import errorLoggerMiddleware from '@middy/error-logger'
import inputOutputLoggerMiddleware from '@middy/input-output-logger'
import httpContentNegotiationMiddleware from '@middy/http-content-negotiation'
import httpContentEncodingMiddleware from '@middy/http-content-encoding'
import httpCorsMiddleware from '@middy/http-cors'
import httpErrorHandlerMiddleware from '@middy/http-error-handler'
import httpEventNormalizerMiddleware from '@middy/http-event-normalizer' // required
import httpHeaderNormalizerMiddleware from '@middy/http-header-normalizer'
import httpJsonBodyParserMiddleware from '@middy/http-json-body-parser'
import httpMultipartBodyParserMiddleware from '@middy/http-multipart-body-parser'
import httpPartialResponseMiddleware from '@middy/http-partial-response'
import httpResponseSerializerMiddleware from '@middy/http-response-serializer'
import httpSecurityHeadersMiddleware from '@middy/http-security-headers'
import httpUrlencodeBodyParserMiddleware from '@middy/http-urlencode-body-parser'
import httpUrlencodePathParametersParserMiddleware from '@middy/http-urlencode-path-parser'
import validatorMiddleware from 'validator' // or `middy-ajv`
import warmupMiddleware from 'warmup'

import eventSchema from './eventSchema.json' assert { type: 'json' }
import responseSchema from './responseSchema.json' assert { type: 'json' }

export const handler = middy({
  timeoutEarlyResponse: () => {
    return {
      statusCode: 408
    }
  }
})
  .use(warmupMiddleware())
  .use(httpEventNormalizerMiddleware())
  .use(httpHeaderNormalizerMiddleware())
  .use(
    httpContentNegotiationMiddleware({
      availableLanguages: ['en-CA', 'fr-CA'],
      availableMediaTypes: ['application/json']
    })
  )
  .use(httpUrlencodePathParametersParserMiddleware())
  // Start oneOf
  .use(httpUrlencodeBodyParserMiddleware())
  .use(httpJsonBodyParserMiddleware())
  .use(httpMultipartBodyParserMiddleware())
  // End oneOf
  .use(httpSecurityHeadersMiddleware())
  .use(httpCorsMiddleware())
  .use(httpContentEncodingMiddleware())
  .use(
    httpResponseSerializerMiddleware({
      serializers: [
        {
          regex: /^application\/json$/,
          serializer: ({ body }) => JSON.stringify(body)
        }
      ],
      default: 'application/json'
    })
  )
  .use(httpPartialResponseMiddleware())
  .use(validatorMiddleware({ eventSchema, responseSchema }))
  .use(httpErrorHandlerMiddleware())
  .handler((event, context, { signal }) => {
    // ...
  })
```
