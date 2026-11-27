# Joy Inbox — AWS-native migration

## Non-negotiable storage rule
- DynamoDB is the system of record for conversations, messages, contacts, quick replies, send idempotency records, greeting settings, push subscriptions, and deletion markers.
- S3 is the only media object store for inbound/outbound images, videos, audio, and documents. Keep the bucket private; use short-lived presigned URLs and server-side encryption.
- Do not use Firestore or Firebase Storage for application data or media.

## Firebase removal scope
- Replace Firebase Admin ID-token verification with Amazon Cognito User Pool JWT verification.
- Replace Firestore listeners/queries/writes with authenticated Lambda API routes backed by DynamoDB. Frontend can poll for updates until a push/realtime channel is deliberately implemented.
- Replace Firebase Cloud Messaging with browser Push API subscriptions (VAPID/Web Push), stored in DynamoDB and delivered by the backend. No FCM SDK/service worker.
- Remove Firebase client SDK, Firebase Admin SDK, Firebase config, Firebase rules/index files, and Firebase environment variables after all references are migrated and builds pass.
- Keep no Firebase dependency in the final app unless a separately approved exception is recorded.

## AWS service design
- Lambda (Node.js 22): API, WhatsApp Cloud API integration, Meta webhook, authorization, media orchestration.
- DynamoDB: single-table design in the existing JoyInboxMessages table where feasible; PK/SK string keys. Define and document exact key patterns and any required GSIs before deployment. Do not assume the existing table has indexes that have not been verified.
- S3: private media bucket, SSE enabled, bucket policy restricted to the Lambda execution role, lifecycle policy only after the retention period is explicitly confirmed.
- Cognito User Pool: email/password owner sign-in; Google federation is a separate configuration step and must not be claimed configured until verified.
- IAM: least-privilege Lambda role for the exact DynamoDB table and S3 bucket.
- Lambda Function URL: public invocation is needed for Meta webhook; application routes must independently validate Cognito JWTs. Configure exact CORS origins in code and Function URL CORS, not wildcard credentials.

## Safety and migration rules
- Work on branch `aws-native-migration`; do not overwrite `main` or deploy an incomplete migration.
- Do not delete or overwrite the existing Lambda deployment while the replacement is incomplete. The currently deployed ZIP is not accepted as production-ready because it still contains Firebase-backed code.
- Do not add placeholder credentials or ask the owner to paste secrets into chat. Use Lambda environment variables or Secrets Manager, entered directly in AWS Console.
- Run backend tests and frontend production build before merging. Then deploy a new ZIP and verify safe health/auth/webhook tests before enabling production traffic.
- Keep existing customer data untouched. A data migration/import plan is required if any Firestore data already exists and must be retained.

## Current verified source gaps (from repository inspection)
- Backend uses `firebase-admin` and Firestore in the API, webhook, push-token, and message persistence paths.
- Backend media storage uses Firebase Storage.
- Frontend uses Firebase Auth, Firestore listeners and writes, and Firebase Messaging.
- Therefore the recently deployed Lambda ZIP is not an AWS-native backend and must not be treated as final.
- The existing Function URL was observed with AWS_IAM auth; that is incompatible with direct Meta webhook delivery and the current browser Bearer-token API pattern. Reconfigure only after Cognito JWT validation is implemented and tested.

## Completion checklist
- [ ] Cognito sign-in and JWT verification
- [ ] DynamoDB repository layer and table/index schema
- [ ] S3 upload, download, delete, presigned URL, and lifecycle policy
- [ ] WhatsApp inbound/outbound persistence migrated to DynamoDB/S3
- [ ] Contacts, quick replies, chat state, greeting settings, and send idempotency migrated
- [ ] Firebase Messaging replaced with Web Push or explicitly removed
- [ ] All frontend Firebase imports and config removed
- [ ] Firebase dependencies and Firebase-only files/rules removed
- [ ] Backend tests and frontend build pass
- [ ] Fresh versioned Lambda ZIP generated from this branch
- [ ] AWS resources/IAM/env verified
- [ ] Lambda deployed and safe integration tests pass
