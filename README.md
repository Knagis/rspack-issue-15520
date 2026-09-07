https://github.com/web-infra-dev/rspack/issues/15520

# rspack `css/module`: `@value` not substituted in `@media` conditions and selectors

Rspack's native CSS modules implementation does not substitute `@value` references
in at-rule preludes (`@media` conditions) and in selector positions, while webpack's
native CSS support (`experiments.css`) substitutes both. Declaration values are
substituted correctly by both bundlers. Rspack emits the invalid CSS silently, with
no warning or error.

## Reproduce

```sh
npm install
npm run build
```

Builds `src/index.js` with webpack 5.108.4 and rspack 2.2.2 using identical configs
(`experiments: { css: true }`, single rule `{ test: /\.css$/, type: "css/module" }`,
no loaders) and prints both emitted stylesheets.

## Input CSS

```css
@value mq: (max-width: 599px);
@value selector: .globalThing;

@media mq {
    .inMedia { color: red; }
}

selector { color: blue; }
```

## Webpack output

```css
@media (max-width: 599px) {
    .index_css-inMedia { color: red; }
}

.globalThing { color: blue; }
```

## Rspack output

```css
@media mq {
    .index_css-inMedia { color: red; }
}

selector { color: blue; }
```

`mq` and `selector` are left verbatim: `@media mq` never matches, and `selector`
becomes an element selector for a nonexistent `<selector>` tag. css-loader (7.1.4)
also substitutes `@media` conditions, so rspack is the outlier among the existing
implementations.
