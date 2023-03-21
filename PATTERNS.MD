# Obligatory Patterns

Fran Aguilera
<fraguile@microsoft.com>

The following document contains recommended development patterns, including samples, for an enterprise-grade AADB2C frontend application.

---

## Shamelessly use [jQuery 3.5.1](https://learn.microsoft.com/en-us/azure/active-directory-b2c/page-layout#jquery-and-handlebars-versions) everywhere

It's not likely AADB2C will stop using it anytime soon. If this becomes a concern, you can always:

- Add it manually
- Add a Polyfill
- Add your own wrappers around jQuery methods.

---

## Sanitize dangerous code

```js
import DOMPurify from 'dompurify';

const purifiedItem = DOMPurify.sanitize(sessionStorage.getItem('sessionItem'));
```

Any kind of input from users or data accessible by the browser should be sanitized to prevent XSS or malicious code.

---

## Template HTML

Separate, meaningfully-named files for any custom-injected HTML into a _components_ directory consisting of templates.

```js
import i18next from 'i18next';
import Handlebars from 'handlebars';

const title = i18next.t('title');
const description = i18next.t('description');

// Fetch and compile Handlebars template
const source = await $.get('template.handlebars');
const template = Handlebars.compile(source);

const html = template({ title, description });
$('#api').html(html);
```

See [the project structure section under configuration](CONFIGURATION.MD#project-structure) for notes on setting up [handlebars](https://handlebarsjs.com/).

---

## Modularize Styles

Separate, meaningfully-named [CSS-modules](https://github.com/css-modules/css-modules) for any injected css.

For any Custom-injected or B2C-injected components, have a corresponding CSS module that styles it into the look and feel of your application.

```js
import styles from './styles.module.css';
$('myElement').addClass(styles['my-style']);
```

See [the webpack section under configuration](CONFIGURATION.md#recommended-bundler) for notes on setting up CSS modules.

See [the project structure section under configuration](CONFIGURATION.MD#project-structure) for notes on where to maintain these CSS modules.

---

## Localization

Perform all localization using i18n in the JavaScript rather than AADB2C policies.
Localizations can be acquired from a remote path if bundle size is an issue:

```js
import i18next from 'i18next';
import HttpBackend from 'i18next-http-backend';

await i18next.use(HttpBackend).init({
  lng: 'en',
  fallbackLng: 'en',
  backend: {
    loadPath: '/locales/{{lng}}/{{ns}}.json',
  },
});
```

It's easier to manage and update. Language changes are reflected in the UX in realtime for users without messing with the flow.
Inject localized content into HTML templates through JavaScript.

---

## General dev patterns

Use modern [ES2022/ESNext](https://dev.to/jasmin/whats-new-in-es2022-1de6) syntax (It can always be transpiled down for compatibility)

Use descriptive function names.

```js
// Bad
function fn() {
  // ...
}
```

```js
// Good
function calculateTotalPrice(items) {
  // ...
}
```

Use camelCasing for variables, functions, and object properties.

```js
const myConst;
let myLet;
var myVar;
const myObj = {
  someProperty: '',
};
```

Use $camelCase syntax to differentiate [jQuery](https://api.jquery.com/category/version/3.5/) variables.

```js
const $myJConst;
```

Use classic function syntax for top-level functions.

```js
function initialize() {
  // ...
}
```

Use arrow syntax for embedded functions.

```js
function initialize() {
  const initializeCallback = () => {};
}
```

Maintain separation of concerns.

- For Ex: code that performs frontend validation should not be mixed with the code that shows an error message due to that validation.

Use common code, but don't keep everything in one place. Most of the code in the project should be in reusable modules that are imported in the JS for each page.

- For Ex: error handlers can all be modularized and exported from an ErrorHandlers module.

Respect formatting rules. Ideally, force formatting on save.

See [the Prettier section under configuration](CONFIGURATION.md#recommended-formatter) for notes on setting up [Prettier](https://prettier.io/) formatting.

---

## Reactive patterns

AADB2C has a frontend framework that is running in the JavaScript of the browser. This framework is in charge of performing a lot of actions on behalf of the policies. One of the core actions is injecting HTML into the `#api` div.

Use the following patterns to perform actions after AADB2C performs injection or to override behavior.

The following patterns are in decreasing order of maintainability.
Patterns towards the top are more maintainable if the AADB2C frontend framework changes. Take difficulty or lengthiness of a solution into account when choosing a pattern.

1. DOM Observers are useful for observing DOM changes that AADB2C makes:

```js
/**
 *  Use a common DOM Mutation Observer where possible.
 */

if (typeof jQuery !== 'undefined' && typeof $ !== 'undefined') {
  $.fn.lazyLoaded = function (callback) {
    if (!callback) {
      return;
    }

    const hasChildren = this.children().length > 0;
    if (hasChildren) {
      callback();
    } else {
      const observer = new MutationObserver(mutations => {
        if (!mutations) {
          return;
        }

        const childAdded = mutations.some(
          mutation =>
            mutation.addedNodes.length > 0 && mutation.target === this[0],
          this
        );

        if (childAdded) {
          observer.disconnect();
          callback();
        }
      });

      observer.observe(this[0], { childList: true });
    }
  };
}
```

2. Reacting to user input. Handler ordering is useful for performing frontend input validation:

```js
/**
 * Create input handlers and change the ordering of handlers to control flow
 * when doing things like input validation.
 */

const $myButton = $('#myButton');
function handleSignupValidation(event) {
  if ($someInput.val() !== $someOtherInput.val()) {
    // Raise relevant events when performing frontend validation.
    $myButton.trigger('signupValidationFailed', ['EmailsDontMatch']);
    // ...
    // Use propagation functions to stop B2C actions when performing validation, etc.
    event.stopImmediatePropagation();
  }
}
$myButton.on(eventName, handleSignupValidation);
// Get all of the handlers
const handlers = $._data($myButton[0], 'events')[eventName.split('.')[0]];
// Get our handler.
const handler = handlers.pop();
// Insert it into the correct position, front in our case, to perform validation.
handlers.unshift(handler);

/**
 * Modularize the code - keep related functionality in separate, meaningfully-named, top-level functions.
 * Ex: don't mix code that's changing the UX with code that's performing validation...
 */

function handleSignupValidationFailedUX(event, data) {
  // ...
  // Inject error template into the UX.
  // ...
}
$myButton.on('signupValidationFailed', handleSignupValidationFailedUX);
```

3. Reacting to network requests (Useful for error handling).

```js
/**
 * Use global AJAX handlers for more complex logic.
 */

function registerGlobalAjaxHandler(ajaxEvent, handler) {
  if (!handler) {
    return;
  }

  if (typeof $(document)[ajaxEvent] !== 'function') {
    return;
  }

  $(document)[ajaxEvent]((event, xhr, settings) => {
    const currentHref = window?.location?.href ?? '';
    if (!currentHref) {
      return;
    }
    const currentURL = new URL(currentHref);

    let destinationHref = settings?.url ?? '';
    if (!destinationHref) {
      return;
    }
    if (/^(\/|\.\/)/.test(destinationHref)) {
      destinationHref = currentURL.origin + destinationHref;
    }
    const requestURL = new URL(destinationHref);

    handler(event, xhr, settings, currentURL, requestURL);
  });
}

function registerAjaxCompleteHandler(handler) {
  registerGlobalAjaxHandler('ajaxComplete', handler);
}

const handleSomeError = registerAjaxCompleteHandler(
  (_event, xhr, _settings, currentURL, requestURL) => {
    const hostsMatch = requestURL.host === currentURL.host;
    const pathInvoked = requestURL.pathname.toLowerCase().endsWith('/somepath');
    const errorReturned =
      xhr?.responseJSON?.errorCode?.toLowerCase() === 'someerrorcode';
    if (hostsMatch && pathInvoked && errorReturned) {
      // ...
    }
  }
);
```

4. Global AADB2C data:

```js
/**
 * Certain data can be accessed from Global objects like SA_FIELDS, CONTENT, SETTINGS, and other objects.
 */

window.CONTENT['required_field'];
```

5. Function overriding:

```js
/**
 * For anything that is more difficult with the above methods, fall-back to
 * function overriding. Find the AADB2C function to override, override the behavior and invoke the original.
 * Example: Want to do something before AADB2C redirects a user.
 */

const b2cRedirect = window.$i2e.redirectToServer;
if (b2cRedirect) {
  window.$i2e.redirectToServer = param => {
    // ...
    // Perform some action before redirecting.
    // Invoke B2C redirect function.
    b2cRedirect.apply(this, arguments);
  };
}
```
