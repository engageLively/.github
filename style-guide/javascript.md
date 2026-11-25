# JavaScript and TypeScript

Follow the community conventions that ESLint's recommended rules and Prettier enforce. The [shared principles](code.md) say what to document; this page says where. Where a project uses TypeScript, types go in the type system and comments carry only meaning.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Variables, functions, methods | `camelCase` | `nextRun`, `fetchReportData` |
| Classes, components, types | `PascalCase` | `ReportSummary`, `HealthPanel` |
| Constants that never change | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Files | follow the framework's convention; otherwise `kebab-case.js` | `report-summary.js` |

## Documentation

Use JSDoc comments placed immediately before each exported function, class and module. The summary is the description. `@param` and `@returns` cover the Results, and `@throws` covers errors. JSDoc has no standard tag for side effects, so write a `Side effects:` paragraph in the description.

```js
/**
 * Upload an instrument export to the server, replacing any earlier upload of the
 * same file.
 *
 * Side effects: overwrites the stored object for this file ID and clears
 * the server's cached summary for its project, so dashboards recompute on the
 * next view.
 *
 * @param {string} fileId - Server identifier of the export, not its filename.
 * @param {Blob} contents - Raw export bytes, as read from disk.
 * @returns {Promise<UploadReceipt>} Resolves when the server has stored the file.
 * @throws {QuotaError} The project's storage quota would be exceeded.
 */
export async function uploadExport(fileId, contents) { ... }
```

Editors and TypeScript read JSDoc, so these comments show up wherever the function is used.

## Tools

- **ESLint** (recommended config plus `eslint-plugin-jsdoc`'s `require-jsdoc` rule for exported functions) and **Prettier** in CI.
- Tests use the framework's standard runner (Vitest, Jest or `node:test`), with tests next to the module or in `tests/`. Group by module or feature, following the repository's layout.
