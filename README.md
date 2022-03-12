# styler
CSS tool that creates atomic styles and works with inline too for SSR'ing emails

# to Install

Uh this doesn't exist on npm yet


# To use it right now

### Install it

Copy the code from src into your library

### Make your styler instance

You need to make your own instance of styler.
The main reason to do this is so you can define the types of what CSS you find acceptable for your project.
I practical example is disallowing float, z-index, etc.
Another common thing is only allowing padding and margins within your design system. Maybe you only allow 2px, 8px, 12px, 16px, and 20px padding.

Another thing you can do is setup transforms. Maybe engineers like taking in px's for font-size, but realistically you want to use rems for accessibility.

Maybe you want to automatically expand something like "padding: 2px" to all 4 top, left, bottom, and right individual fields.


```typescript
import Styler from "lib/styler";

import type {Styles} from "./CSSStyles";
import expansion from "./transforms/expansion";
import fixPixelRemNumbers from "./transforms/fixPixelRemNumbers";

// This file is the instance of styler, you'd want one per config, probably one per site
const styler = new Styler<Styles>({
  production: false, // if true, we don't do hashing at all.
  inline: false, // if true, we use style instead of className (for emails)
  transforms: [expansion, fixPixelRemNumbers], // array of ordered transforms that change input to a different output
});

export default styler;
export const cx = styler.cx;
```


### Make a Theme provider

Imagine you are using Next.js and this is your _app.tsx.

You can actually inject themes that override themes, which is cool, so you can have subtrees be different themes

```typescript
import Head from "next/head";
import React from "react";
import ThemeRoot from "styler/ThemeRoot";
import type {AppProps, AppContext} from "next/app";
import {cssVariables} from "myinstance/Theme";

function MyApp({Component, pageProps}: AppProps) {
  return (
    <ThemeRoot variables={cssVariables}>
      <Head>
        <link rel="icon" href="/favicon.png" />
      </Head>
      <Component {...pageProps} />
    </ThemeRoot>
  );
}

export default MyApp;
```


### Create some styles

```Button.tsx
import styler from "myinstance/styler"

// this will create a container element
const ButtonImpl = styler.button({
  border: '1px solid black',
  backgroundColor: 'lightgray',
  ':hover': {
    backgroundColor: 'slategray',
  }
});

// this creates mixable styles to any component
const styles = styler.create({
  blue: {
    backgroundColor: 'var(--blue-color)',
    ':hover': {
      backgroundColor: 'var(--darkblue-color)',
    }
  }
  red: {
    backgroundColor: 'var(--red-color)',
    ':hover': {
      backgroundColor: 'var(--darkred-color)',
    }
  }
});

export default function Button({children, onClick, design}) {
  // the order applied here in cx determines the style precedence, like an object merge
  return (
    <ButtonImpl
      cx={[design === "blue" && styles.blue, design === "red" && styles.red]}
      onClick={onClick}
    >
      {children}
    </ButtonImpl>
  );
}
```



### Inline CSS

Tbh right now my email components are actually different components, because I used styler.inline,
but we could do a context


```typescript

import StylerInlineContext from "myinstance/styler";

import EmailLayout from "layouts/EmailLayout";
import Button from "components/Button";
import Heading from "components/Heading";

interface Props {
  user: any;
}
export default function WelcomeEmail({user}: Props) {
   // StylerInlineContext.Provider could signal to the renderer to not apply as className but just merge into styles
   return (
     <StylerInlineContext.Provider value={true}>
       <EmailLayout>
          <Heading>Welcome to Styler, {user.name}</Heading>
          <Button design="blue" href="http://mywebsite.com/login">Login</Button>
       </EmailLayout>
     </StylerInlineContext.Provider>
   );
}

```
