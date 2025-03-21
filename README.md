# TypeORM Paginator

<p>
  <a href="https://npmcharts.com/compare/typeorm-cursor-paginate?minimal=true"><img alt="Downloads" src="https://img.shields.io/npm/dt/typeorm-cursor-paginate.svg?style=flat-square" /></a>
  <a href="https://www.npmjs.com/package/typeorm-cursor-paginate"><img alt="License" src="https://img.shields.io/npm/l/typeorm-cursor-paginate.svg?style=flat-square" /></a>
</p>

Cursor-based pagination that works with [TypeORM Query Builder](https://typeorm.io/#/select-query-builder). Read about the general idea of cursor-based pagination [here](https://jsonapi.org/profiles/ethanresnick/cursor-pagination/).

This package is a fork of the [typeorm-paginator](https://www.npmjs.com/package/typeorm-paginator) package with some tweaks. See the [Key differences from `typeorm-paginator`](#key-differences-from-typeorm-paginator) section for more details.

The biggest difference is **directional** cursors. Directional cursors store the direction of pagination inside them. They allow to provide only one parameter to paginate in any direction.

- [Installation](#installation)
- [Usage](#usage)
  - [With the Configuration step](#with-the-configuration-step)
    - [Configuration](#configuration)
    - [Retrieving pages](#retrieving-pages)
  - [Without the configuration step](#without-the-configuration-step)
- [Key differences from other packages](#key-differences-from-other-packages)
  - [`typeorm-paginator`](#typeorm-paginator-1)
  - [`typeorm-cursor-pagination`](#typeorm-cursor-pagination)
- [Contributing](#contributing)
- [License](#license)

## Installation

```bash
npm install typeorm-cursor-paginate --save
```

## Usage

### With the Configuration step

Start by importing the `CursorPaginator` class:

```typescript
import { CursorPaginator } from "typeorm-cursor-paginate";
```

#### Configuration

Instantiate the paginator with your target entity and define the ordering strategy. In this example, users are sorted by their `name` in ascending order and by `id` in descending order:

```typescript
const paginator = new CursorPaginator(User, {
  orderBy: { name: "ASC", id: "DESC" },
});
```

Prepare your query:

```typescript
const query = repoUsers.createQueryBuilder();
// you can apply desired "where" conditions to the query
```

#### Retrieving pages

1. Retrieving the First Page

    Use the paginator to fetch the initial set of results. Here, a limit of 2 items per page is specified:

    ```typescript
    const firstPageResult = await paginator.paginate(query, { limit: 2 });
    ```

    Output structure:

    ```typescript
    {
      nodes: [
        User { id: 4, name: 'a' },
        User { id: 3, name: 'b' },
      ],
      hasPrevPage: false,
      hasNextPage: true,
      prevPageCursor: "some-cursor-string",
      nextPageCursor: "some-cursor-string",
    }
    ```

2. Navigating to the Next Page

    To retrieve the next set of results, pass the `nextPageCursor` from the first query:

    ```typescript
    const secondPageResult = await paginator.paginate(query, {
      limit: 2,
      // Use the nextPageCursor from the previous result
      pageCursor: firstPageResult.nextPageCursor,
    });
    ```

    Output structure:

    ```typescript
    {
      nodes: [
        User { id: 1, name: 'c' },
        User { id: 2, name: 'c' },
      ],
      hasPrevPage: true,
      hasNextPage: true,
      prevPageCursor: "some-cursor-string",
      nextPageCursor: "some-cursor-string",
    }
    ```

### Without the configuration step

There is also a way to skip the configuration step and use the `paginate` function directly. First, import the default from the package:

```typescript
import paginate from "typeorm-cursor-paginate";
```

Then, use it like this:

```typescript
// create a query builder
const query = repoUsers.createQueryBuilder();
// you can apply desired "where" conditions to the query

// Get the first page
const result = await paginate(User, query, {
  orderBy: { name: "ASC", id: "DESC" },
  limit: 2,
});

// Get the next page
const resultNext = await paginate(User, query, {
  orderBy: { name: "ASC", id: "DESC" },
  limit: 2,
  pageCursor: result.nextPageCursor,
});
```

## Key differences from other packages

### `typeorm-paginator`

[typeorm-paginator](https://www.npmjs.com/package/typeorm-paginator) is the original package that this one is based on.

Here are the key differences:

- **Directional cursors**: In the original package, cursors are not directional. The pagination direction was determined by what argument the cursor is passed to.
  This package stores the direction of pagination inside the cursor. It allows to provide only one parameter to paginate in any direction.
- **Type safety**: Added some more type safety to the code. Now the `orderBy` property only accepts keys that are present in the entity.
- **Removed PageCursor**: The `PageCursor` class was removed. This package is about cursor-based pagination.
- **No default "limit"**: Original package had a default limit of 20.Now, if the limit is omitted, all results will be returned.

### `typeorm-cursor-pagination`

[typeorm-cursor-pagination](https://www.npmjs.com/package/typeorm-cursor-pagination) is another package that provides cursor-based pagination for TypeORM. As of now, March 2025, it gets more and more popular than the `typeorm-paginator` package.

Here are the key differences:

- **Can provide different directions for different columns in orderBy**: In the original package, all columns in the `orderBy` array had to have the same direction. This package allows to provide different directions for different columns.
- **Directional cursors**: see above
- **No default "limit"**: see above
- **Custom transformation of the cursor**: In the current package, custom transformers to stringify and parse the cursor can be provided. This feature comes from the original package (`typeorm-paginator`).

## Contributing

All contributions are welcome, open a pull request or issue any time.

_Please try to commit your changes using a descriptive commit message._

## License

Released under the [MIT License](https://github.com/Oriery/typeorm-cursor-paginate/blob/main/License)
