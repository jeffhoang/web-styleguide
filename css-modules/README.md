# Cisco Spark CSS Style Guide

*A mostly reasonable approach to CSS-in-JavaScript

## Table of Contents

1. [Naming](#naming)
1. [Ordering](#ordering)
1. [Nesting](#nesting)
1. [Inline](#inline)
1. [Themes](#themes)

## Naming

  - Use camelCase for object keys (i.e. "selectors").

    > Why? We access these keys as properties on the `styles` object in the component, so it is most convenient to use camelCase.

    ```js
    // bad
    {
      'bermuda-triangle': {
        display: 'none',
      },
    }

    // good
    {
      bermudaTriangle: {
        display: 'none',
      },
    }
    ```

  - Use an underscore for modifiers to other styles.

    > Why? Similar to BEM, this naming convention makes it clear that the styles are intended to modify the element preceded by the underscore. Underscores do not need to be quoted, so they are preferred over other characters, such as dashes.

    ```js
    // bad
    {
      bruceBanner: {
        color: 'pink',
        transition: 'color 10s',
      },

      bruceBannerTheHulk: {
        color: 'green',
      },
    }

    // good
    {
      bruceBanner: {
        color: 'pink',
        transition: 'color 10s',
      },

      bruceBanner_theHulk: {
        color: 'green',
      },
    }
    ```

  - Use `selectorName_fallback` for sets of fallback styles.

    > Why? Similar to modifiers, keeping the naming consistent helps reveal the relationship of these styles to the styles that override them in more adequate browsers.

    ```js
    // bad
    {
      muscles: {
        display: 'flex',
      },

      muscles_sadBears: {
        width: '100%',
      },
    }

    // good
    {
      muscles: {
        display: 'flex',
      },

      muscles_fallback: {
        width: '100%',
      },
    }
    ```

  - Use a separate selector for sets of fallback styles.

    > Why? Keeping fallback styles contained in a separate object clarifies their purpose, which improves readability.

    ```js
    // bad
    {
      muscles: {
        display: 'flex',
      },

      left: {
        flexGrow: 1,
        display: 'inline-block',
      },

      right: {
        display: 'inline-block',
      },
    }

    // good
    {
      muscles: {
        display: 'flex',
      },

      left: {
        flexGrow: 1,
      },

      left_fallback: {
        display: 'inline-block',
      },

      right_fallback: {
        display: 'inline-block',
      },
    }
    ```

  - Use device-agnostic names (e.g. "small", "medium", and "large") to name media query breakpoints.

    > Why? Commonly used names like "phone", "tablet", and "desktop" do not match the characteristics of the devices in the real world. Using these names sets the wrong expectations.

    ```js
    // bad
    const breakpoints = {
      mobile: '@media (max-width: 639px)',
      tablet: '@media (max-width: 1047px)',
      desktop: '@media (min-width: 1048px)',
    };

    // good
    const breakpoints = {
      small: '@media (max-width: 639px)',
      medium: '@media (max-width: 1047px)',
      large: '@media (min-width: 1048px)',
    };
    ```

## Ordering

  - Define styles after the component.

    > Why? We use a higher-order component to theme our styles, which is naturally used after the component definition. Passing the styles object directly to this function reduces indirection.

    ```jsx
    // bad
    const styles = {
      container: {
        display: 'inline-block',
      },
    };

    function MyComponent({ styles }) {
      return (
        <div {...css(styles.container)}>
          Never doubt that a small group of thoughtful, committed citizens can
          change the world. Indeed, it’s the only thing that ever has.
        </div>
      );
    }

    export default withStyles(() => styles)(MyComponent);

    // good
    function MyComponent({ styles }) {
      return (
        <div {...css(styles.container)}>
          Never doubt that a small group of thoughtful, committed citizens can
          change the world. Indeed, it’s the only thing that ever has.
        </div>
      );
    }

    export default withStyles(() => ({
      container: {
        display: 'inline-block',
      },
    }))(MyComponent);
    ```

## Nesting

  - Leave a blank line between adjacent blocks at the same indentation level.

    > Why? The whitespace improves readability and reduces the likelihood of merge conflicts.

    ```js
    // bad
    {
      bigBang: {
        display: 'inline-block',
        '::before': {
          content: "''",
        },
      },
      universe: {
        border: 'none',
      },
    }

    // good
    {
      bigBang: {
        display: 'inline-block',

        '::before': {
          content: "''",
        },
      },

      universe: {
        border: 'none',
      },
    }
    ```

---

CSS puns adapted from [Saijo George](http://saijogeorge.com/css-puns/).