# AWS Lambda deployment package

The Lambda handler for this backend is `index.handler`.

## Build the current deployment ZIP

The GitHub Actions workflow `.github/workflows/build-lambda-deployment.yml` builds a ZIP from the current `backend/` source, installs production dependencies, and runs `npm test` before publishing the artifact.

1. Open the repository's **Actions** tab.
2. Open **Build AWS Lambda deployment ZIP**.
3. Open the successful run for the latest commit.
4. Download the artifact named `joy-inbox-lambda-<commit-sha>`.
5. In AWS Lambda, open the `joy-inbox-api` function and upload that ZIP under **Code > Upload from > .zip file**.

The ZIP root must contain `index.js`, `src/`, and `node_modules/` (not an extra enclosing `backend/` directory).

## Lambda settings

- Runtime: Node.js 22.x
- Handler: `index.handler`
- Architecture: x86_64 is supported by this package.

Do not upload the old root-level `joy-inbox-lambda.zip` unless its contents have been independently verified against the latest source.

## Important configuration note

The current `backend/src/media.js` implementation uses Firebase Storage. This ZIP build does not migrate media storage to Amazon S3. S3 migration, bucket permissions, environment variables, and live Lambda deployment must be completed and tested separately.
