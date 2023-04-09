
```sh
━━━━━━━━━┏┓━━━━━━┏━┓━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
━━━━━━━━━┃┃━━━━━━┃┏┛━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┏┓━━━━━━
┏┓┏┓┏┓━┏┓┃┃━━━┏┓┏┛┗┓┏━━┓━━┏━━┓━┏┓━━┏━━┓┏━┓┏━━┓┏━┓━━┗┛┏━━┓━━
┃┗┛┃┃┃━┃┃┃┃━┏┓┣┫┗┓┏┛┃┏┓┃━━┗━┓┃━┣┫━━┃┏━┛┃┏┛┃┏┓┃┃┏┓┓━┏┓┃┏┓┃━━
┃┃┃┃┃┗━┛┃┃┗━┛┃┃┃━┃┃━┃┃━┫┏┓┃┗┛┗┓┃┃┏┓┃┗━┓┃┃━┃┗┛┃┃┃┃┃━┃┃┃┃━┫┏┓
┗┻┻┛┗━┓┏┛┗━━━┛┗┛━┗┛━┗━━┛┗┛┗━━━┛┗┛┗┛┗━━┛┗┛━┗━━┛┗┛┗┛━┃┃┗━━┛┗┛
━━━━┏━┛┃━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┏┛┃━━━━━━
━━━━┗━━┛━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┗━┛━━━━━━
```
# Custom Chat bot based on ChatGPT

There are quite a few chat bots available online. However, most of them are based on a pre-trained model. This means that the chat bot is limited to the topics that the model was trained on. This is not the case with this chat bot. This chat bot is based on a custom model that was trained on a dataset of my own creation. The dataset consists of 1000+ conversations and journal entries and short stories and some experiences. 
I plan to train the model with this dataset for 1000 epochs end then to fine tune until I'm happy. I will then deploy this web-app using SveltKit or maybe Streamlit

## Setting up the new app

I will be creating the app with sveltekit. I will be using the following command to create the app:

```bash
npm create svelte@latest .
```

### Then to setup Tailwind CSS

I will be using the following command:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Install tailwindcss and its peer dependencies, then generate your `tailwind.config.js` and `postcss.config.js` files.

### Enable use of PostCSS in `<style>` blocks

In your `svelte.config.js` file, import vitePreprocess to enable processing `<style>` blocks as PostCSS.

```js
# svelte.config.js
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/kit/vite';

/** @type {import('@sveltejs/kit').Config} */
const config = {
  kit: {
    adapter: adapter()
  },
  preprocess: vitePreprocess()
};

export default config;
```

### Configure your template paths

Add the paths to all of your template files in your `tailwind.config.js` file.

```js
# tailwind.config.js

/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{html,js,svelte,ts}'],
  theme: {
    extend: {}
  },
  plugins: []
};
```

### Add the Tailwind directives to your CSS

Create a `./src/app.css` file and add the @tailwind directives for each of Tailwind’s layers.

```css
# app.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Import the CSS file

Create a `./src/routes/+layout.svelte` file and import the newly-created `app.css` file.

```svelte
# +layout.svelte
<script>
  import "../app.css";
</script>

<slot />
```

### Start your build process

Run your build process with npm run dev.

```bash
npm run dev
```

### Start using Tailwind in your project

Start using Tailwind’s utility classes to style your content, making sure to set `lang="postcss"` for any `<style>` blocks that need to be processed by Tailwind.

```svelte
# +page.svelte
<h1 class="text-3xl font-bold underline">
  Hello world!
</h1>

<style lang="postcss">
  :global(html) {
    background-color: theme(colors.gray.100);
  }
</style>
```

## API boilerplate

The first part will be setting up the API calls, making sure we can call our own server, and then from the server making sure we can call the OpenAi chat completion API endpoint.

To make our server, we create a file our file like this:

```ts
// routes/api/chat/+server.ts

import { json, type RequestEvent } from '@sveltejs/kit';
import { OPENAI_API_SECRET_KEY } from '$env/static/private';

export const POST = async (event: RequestEvent) => {
    const requestBody = await event.request.json();
    const { message: _message } = requestBody;

    /**
     * Request config
     */
    const completionHeaders = {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${OPENAI_API_SECRET_KEY}`
    };
    const messages = [
        {
            role: 'system',
            content: 'You are a Alfred, a most helpful and loyal fictional butler to Batman.'
        }
        // { role: 'user', content: 'Alfred, where did I leave my batmobile?' },
        // {
        //     role: 'assistant',
        //     content:
        //         'Sir, you left the Batmobile in the Batcave, where it is normally parked. Shall I have it brought to you?'
        // }
    ];
    const completionBody = {
        model: 'gpt-3.5-turbo',
        messages
    };

    /**
     * API call
     */
    const completion = await fetch('<https://api.openai.com/v1/chat/completions>', {
        method: 'POST',
        headers: completionHeaders,
        body: JSON.stringify(completionBody)
    })
        .then((res) => {
            if (!res.ok) {
                throw new Error(res.statusText);
            }
            return res;
        })
        .then((res) => res.json());

    const message = completion?.choices?.[0]?.message?.content || '';

    return json(message);
};
```

Note the body of the request we send to OpenAi. We use the `gpt-3.5.-turbo` model for chat completion, and the messages are an array of objects with each object having a `role` that can either be `system`, `assistant`, or `user`. More about these roles and the shape of the request and response here:

[https://platform.openai.com/docs/guides/chat/introduction](https://platform.openai.com/docs/guides/chat/introduction)

And to call our server we create a function within our component and call it upon mounting:

```ts
// routes/+page.ts

<script lang="ts">
    import { onMount } from 'svelte';

    onMount(async () => {
        console.log('Hello world!');

        const what = await handleChatCompletion();

        console.log('what: ', what);
    });

    const handleChatCompletion = async () => {
        const response = await fetch('/api/chat', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                message: 'Hello world!'
            })
        }).then((res) => res.json());

        return response;
    };
</script>

<h1 class="text-3xl font-bold underline">Hello world!</h1>
```

All the calls are hard-coded right now, but that’ll do just fine as we just want to make sure the simplest calls are working.

## UI boilerplate

Next, we’ll prettify the UI with `tailwindcss`. I went ahead and bought the tailwind UI package for this. It doesn’t explicitly support svelte but it works well enough.

We add the UI for our chat messages, along with appropriate headshots for profile pics, provided by Midjourney:

```html
// chat-message.svelte
<script lang="ts">
    export let name = '';
    export let message = '';

    const imgSrc = name === 'Alfred' ? '/alfred-headshot.png' : '/batman-headshot.png';
</script>

<li class="flex py-4">
    <img class="h-10 w-10 rounded-full" src={imgSrc} alt="" />
    <div class="ml-3">
        <p class="text-sm font-medium text-gray-900">{name}</p>
        <p class="text-sm text-gray-500">{message}</p>
    </div>
</li>
```

```svelte
// +page.svelte
<script lang="ts">
    import { onMount } from 'svelte';

    import ChatMessage from './chat-message.svelte';

    onMount(async () => {
        const what = await handleChatCompletion();

        console.log('what: ', what);
    });

    const handleChatCompletion = async () => {
        const response = await fetch('/api/chat', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                message: 'Hello world!'
            })
        }).then((res) => res.json());

        return response;
    };
</script>

<ul role="list" class="divide-y divide-gray-200">
    <ChatMessage name="Alfred" message="Hello world!" />

    <ChatMessage name="Batman" message="Hello Alfred!" />
</ul>

<form class="mt-4">
    <div class="relative mt-1 flex items-center">
        <input
            type="text"
            name="search"
            id="search"
            class="block w-full rounded-md border-0 py-1.5 pr-14 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
        />
        <button
            on:click={() => {
                console.log('clicky');
            }}
            type="submit"
            class="absolute inset-y-0 right-0 flex py-1.5 pr-1.5"
        >
            <kbd
                class="inline-flex items-center rounded border border-gray-200 px-1 font-sans text-xs text-gray-400"
                >Enter</kbd
            >
        </button>
    </div>
</form>
```

## Connecting it all together

And finally, we tie everything together, formatting the messages to and from our server.

We want our Alfred bot to remember some past context when responding, so we keep track of these messages with an array and update it upon submitting messages.

```ts
// +page.ts
<script lang="ts">
    import { onMount } from 'svelte';

    import ChatMessage from './chat-message.svelte';

    let messages = [] as any;
    let inputMessage = '';

    onMount(async () => {
        await handleChatCompletion();
    });

    const handleChatCompletion = async () => {
        const userMessage = {
            role: 'user',
            content: inputMessage
        };

        const response = await fetch('/api/chat', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                isInitializing: messages.length === 0,
                priorMessages: messages,
                message: inputMessage
            })
        }).then((res) => res.json());

        if (inputMessage) {
            messages = messages.concat([userMessage]);
        }

        messages = messages.concat(response);
        inputMessage = '';

        return response;
    };
</script>

<ul class="divide-y divide-gray-200">
    {#if messages.length > 0}
        {#each messages as message}
            <ChatMessage role={message.role} message={message.content} />
        {/each}
    {/if}
</ul>

<form class="mt-4">
    <div class="relative mt-1 flex items-center">
        <input
            bind:value={inputMessage}
            type="text"
            name="search"
            id="search"
            class="block w-full rounded-md border-0 py-1.5 pr-14 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
        />
        <button
            on:click={handleChatCompletion}
            type="submit"
            class="absolute inset-y-0 right-0 flex py-1.5 pr-1.5"
        >
            <kbd
                class="inline-flex items-center rounded border border-gray-200 px-1 font-sans text-xs text-gray-400"
                >Enter</kbd
            >
        </button>
    </div>
</form>
```

```svelte
// chat-message.svelte

<script lang="ts">
    export let role = '';
    export let message = '';

    const imgSrc = role === 'assistant' ? '/alfred-headshot.png' : '/batman-headshot.png';
    const name = role === 'assistant' ? 'Alfred' : 'Batman';
</script>

<li class="flex py-4">
    <img class="h-10 w-10 rounded-full" src={imgSrc} alt="" />
    <div class="ml-3">
        <p class="text-sm font-medium text-gray-900">{name}</p>
        <p class="text-sm text-gray-500">{message}</p>
    </div>
</li>

```

```svelte
// routes/api/chat/+server.ts

import { json, type RequestEvent } from '@sveltejs/kit';
import { OPENAI_API_SECRET_KEY } from '$env/static/private';

export const POST = async (event: RequestEvent) => {
    const requestBody = await event.request.json();
    const { priorMessages = [], message, isInitializing = false } = requestBody;

    /**
     * Request config
     *
     * <https://platform.openai.com/docs/guides/chat/introduction>
     *
     * Roles: 'system', 'assistant', 'user'
     */
    const completionHeaders = {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${OPENAI_API_SECRET_KEY}`
    };
    const initialMessage = {
        role: 'system',
        content:
            'You are a Alfred, a most helpful and loyal fictional butler to Batman. Your responses should be directed directly to Batman, as if he were asking you himself.'
    };
    const messages = isInitializing
        ? [initialMessage]
        : [
                initialMessage,
                ...priorMessages,
                {
                    role: 'user',
                    content: message
                }
          ];
    const completionBody = {
        model: 'gpt-3.5-turbo',
        messages
    };

    /**
     * API call
     */
    const completion = await fetch('<https://api.openai.com/v1/chat/completions>', {
        method: 'POST',
        headers: completionHeaders,
        body: JSON.stringify(completionBody)
    })
        .then((res) => {
            if (!res.ok) {
                throw new Error(res.statusText);
            }
            return res;
        })
        .then((res) => res.json());

    const completionMessage = completion?.choices?.map?.((choice) => ({
        role: choice.message?.role,
        content: choice.message?.content
    }));

    return json(completionMessage);
};
```

Thats it for setting up the chatbot!
Now the next step is to start training the model to respond to our messages.


## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
