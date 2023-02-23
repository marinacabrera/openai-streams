# OpenAI Streams

[**Github**](https://github.com/gptlabs/openai-streams) |
[**NPM**](https://npmjs.com/package/nextjs-openai) |
[**Docs**](https://openai-streams.vercel.app)

This library returns OpenAI API responses as streams only. Non-stream endpoints
like `edits` etc. are simply a stream with only one chunk update.

- Prioritizes streams, so you can display a completion as it arrives.
- Auto-loads `OPENAI_API_KEY` from `process.env`.
- Uses the same function for all endpoints, and switches the type based on the
   `OpenAI(endpoint, ...)` signature.

### Installation

```bash
yarn add openai-streams
# -or-
npm i --save openai-streams
```

### Usage

1. **Set the `OPENAI_API_KEY` env variable** (or pass the `{ apiKey }` option).

   The library will throw if it cannot find an API key. Your program will load
   this at runtime from `process.env.OPENAI_API_KEY` by default, but you may
   override this with the `{ apiKey }` option.

   **IMPORTANT:** For security, you should only load this from a `process.env`
   variable.

   ```ts
   await OpenAI(
     "completions", 
     {/* params */}, 
     { apiKey: process.env.MY_SECRET_API_KEY }
   )
   ```

2. **Call the API via `await OpenAI(endpoint, params)`.**

   The `params` type will be inferred based on the `endpoint` you provide, i.e.
   for the `"edits"` endpoint, `import('openai').CreateEditRequest` will be
   enforced.


#### Edge/Browser: Consuming streams in Next.js Edge functions

```ts
import { OpenAI } from "openai-streams";

export default async function handler() {
  const stream = await OpenAI(
    "completions",
    {
      model: "text-davinci-003",
      prompt: "Write a happy sentence.\n\n",
      max_tokens: 100
    },
  );

  return new Response(stream);
}

export const config = {
  runtime: "edge"
};
```


### Node: Consuming streams in Next.js API Route (Node)

If you cannot use an Edge runtime or want to consume Node.js streams for another
reason, use `openai-streams/node`:

```ts
import type { NextApiRequest, NextApiResponse } from "next";
import { OpenAI } from "openai-streams/node";

export default async function test (_: NextApiRequest, res: NextApiResponse) {
  const stream = await OpenAI(
    "completions",
    {
      model: "text-davinci-003",
      prompt: "Write a happy sentence.\n\n",
      max_tokens: 25
    }
  );

  stream.pipe(res);
}
```


<sub>See the example in
[`example/src/pages/api/hello.ts`](https://github.com/gptlabs/openai-streams/blob/master/src/pages/api/hello.ts).</sub>
<sub>See also
[`src/pages/api/demo.ts`](https://github.com/gptlabs/nextjs-openai/blob/master/src/pages/api/demo.ts)
in `nextjs-openai`.</sub>

### Notes

1. Internally, streams are often manipulated using generators via `for await
   (const chunk of yieldStream(stream)) { ... }`. We recommend following this
   pattern if you find it intuitive.