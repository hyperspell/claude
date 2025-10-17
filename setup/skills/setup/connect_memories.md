# Integrate the Hyperspell SDK to let your users connect their accounts.

Before Hyperspell can automatically ingest your user's data, they need to connect their accounts.

To do so, we need to do two things:

1. Initialize the SDK and generate a user token that we can safely pass the the OAuth flow
2. Put a button into your app that opens the Hyperspell Connect page that lets users connect their accounts

Add the following two tasks to your checklist as subtasks of `Add memories`:

```
- [ ] Step 1: Get available integrations
- [ ] Step 2: Initialize the SDK
- [ ] Step 3: Add a Connect button
```

## 1. Get available integrations

Use the API key you set up earlier to get the available integrations for this project:

```sh
curl --request GET \
  --url https://api.hyperspell.com/integrations/list \
  --header 'Authorization: Bearer API_KEY'
```

Note the `provider` name for each integration. 


## 2. Initialize the SDK

When we're initializing the SDK, we will pass a user ID to it to identify the currently logged in user. If there doesn't seem to be any user management, we will use the user id `anonymous` instead.

First, examine the code base to determine how to get the currently logged in user in a server-side action.

Create a typescript file with a server side action in an appropriate directory (ie `src/actions/hyperspell.ts`) and add the following code — modify it to fit the project, and replace `PROVIDERS` with an array of the providers you retrieved in step 1.

```typescript
'use server'

import Hyperspell from 'hyperspell';

export async function getUserToken() {
    const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules

    const hyperspell = new Hyperspell({ apiKey: process.env.HYPERSPELL_API_KEY });
    const response = await hyperspell.auth.userToken({user_id: userId});
    return response.token;
}

export async function search(query: string, answer: boolean = true) {
    const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules
    const hyperspell = new Hyperspell({ apiKey: process.env.HYPERSPELL_API_KEY, userID: userId });

    const response = await hyperspell.memories.search({
        query,
        answer,
        sources = <PROVIDERS>;
    });
    return response.answer;
}
```


## 3. Add a Connect button

Display the following message to your user:

```
We will now add a button to your project that opens Hyperspell Connect. On this page, your users can connect their <PROVIDERS> account. We will use the `userToken` from the code to securely identify your user on this page.
```

Replace `<PROVIDERS>` with an oxford comma separate list of providers we just fetched.

Then, we need to find an appropriate place to place the button. Analyize the codebase to determine if it has any of the following:

- A Settings menu / dropdown on the main page
- A settings page or modal
- An onboarding flow
- A chat UI that has the option to add a custom action or button close to the input area.

Based on what you find, offer the user a multiple choice menu to ask where to put the button (and offer other that lets the user describe it themselves.)

After you determined where to put the button, find the file that contains the component which should contain the button. In this file, import our `getUserToken()` action, ie with

```typescript
import { getUserToken } from '@/actions/hyperspell';
```

(Modify the import path depending on where you put the file).

in the component that renders the button, we need to determine the URL of the current page to use as a redirect url — determine how to use this projects router 

first create the target URL like this:

```typescript
const token = getUserToken();
const connectUrl = `https://connect.hyperspell.com?token=${token}&redirect_uri=${window.location.href};
```

If the project already has built-in button components, use that one. If it has other buttons that are style with ie. tailwind css, copy the style from other buttons. At worst, simply use an `<a>` element and style it yourself. As a `href` use the `connectUrl` we constructed.

Also display the Hyperspell logo as an icon on the button, using the common way this is done in this project. You can use this SVG as the logo (replace the fill color with something appropriate for the button).

```xml
<svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M27.963 25.1327C28.1219 25.409 28.0219 25.7361 27.7881 25.8964C27.6981 25.9581 27.5867 25.9958 27.4639 25.996H4.57721C4.13467 25.9958 3.85733 25.5161 4.07819 25.1327L5.30865 23.0009H26.7325L27.963 25.1327ZM25.4542 20.7831C25.4956 20.8557 25.4898 20.9379 25.4522 21.0009H15.8975C15.865 20.9474 15.854 20.8801 15.877 20.8143C16.0196 20.4085 16.1474 19.9952 16.2598 19.576C16.2961 19.4408 16.1927 19.3089 16.0528 19.3085H7.81061C7.64482 19.3083 7.54145 19.1289 7.62408 18.9852L8.76862 16.9999H23.2725L25.4542 20.7831ZM19.794 11.1835C19.8706 11.1838 19.9422 11.2246 19.9805 11.2909L22.0118 14.8143C22.0466 14.8752 22.0469 14.9424 22.0245 14.9999H16.8467C16.8438 14.5065 16.82 14.0176 16.7764 13.535C16.7666 13.4251 16.6739 13.3411 16.5635 13.3407H11.252C11.0865 13.3405 10.9832 13.1612 11.0655 13.0175L12.0606 11.2909C12.0991 11.2244 12.1704 11.1836 12.2471 11.1835H19.794ZM15.5225 5.28894C15.7438 4.90519 16.2974 4.90516 16.5186 5.28894L18.6592 8.99988H15.7305C15.4488 8.25709 15.1204 7.53696 14.7432 6.84753C14.7072 6.78125 14.7086 6.70104 14.7462 6.63562L15.5225 5.28894Z" fill="#000000"/>
</svg>
```

Finally, display the following message to the user:

```
Great, I've created a button that lets your users connet their accounts. Feel free to try it out right now!
```
