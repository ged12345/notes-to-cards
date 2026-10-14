# Lesson 01 — Project setup, a safe AWS account, and a first React component

**Tag:** `lesson-01` · **Time:** ~1 day · **Builds on:** nothing — this is the start

## Goal
By the end of this lesson you have:
- a GitHub repo with a React + TypeScript app (Vite) that runs locally
- an AWS account you can use from the command line **without long-lived access keys**
- AWS CDK bootstrapped and ready to deploy infrastructure
- a small deck of flashcards you can flip and add to

## The concepts

### Vite
Vite is the dev server and build tool. In development it serves your source files directly as ES modules and converts TypeScript/JSX on the fly, so it starts instantly and hot-reloads on save. `npm run build` bundles everything into a static `dist/` folder — which we'll later host on AWS.

### Signing in to AWS without access keys (IAM Identity Center)
The old way to use AWS from a terminal was an **access key**: a permanent ID and secret stored in `~/.aws/credentials`. If that file leaks, someone has your account until you notice.

**IAM Identity Center** (formerly AWS SSO) replaces that with temporary credentials:
- You create a **user**, put them in a **group** (e.g. `Admins`), and give that group a **permission set** (e.g. `AdministratorAccess`) on your **AWS account**.
- The CLI signs you in through the browser (`aws sso login`) and receives credentials that expire after a few hours.

Users go in groups; groups get permission sets on accounts. That three-way link is called an **assignment**.

### Budget alarm
Before creating anything that costs money, set an AWS Budget (e.g. US$10/month) that emails you when you approach it. It's the seatbelt for everything that follows.

### Region
AWS resources live in a **region**. This project uses **`us-east-1` (N. Virginia)** because new AI models usually arrive there first. The trade-off is extra latency from Australia and the data being stored overseas. Pick one region and use it for everything.

### AWS CDK and bootstrapping
**CDK (Cloud Development Kit)** lets you describe infrastructure — Lambda functions, databases, APIs — in TypeScript. CDK converts that code into a CloudFormation template and deploys it.

`cdk bootstrap` is a one-time setup per account and region. It creates a stack called **CDKToolkit** containing an S3 bucket (where CDK uploads your code and templates) and IAM roles that CDK uses to deploy. You can see it in the CloudFormation console.

### React: state and components
A **component** is a function that returns JSX. **State** (`useState`) is a value React remembers between renders. When you call the setter, React re-runs the component and updates the page to match. You never change the DOM yourself — you change state, and the UI follows.

## Steps

1. **Repo and frontend**
   ```bash
   npm create vite@latest web -- --template react-ts
   cd web && npm install && npm run dev
   ```
   Add a root `.gitignore` covering `node_modules`, `.env*`, `cdk.out` and `dist`.

2. **AWS account**
   - Turn on MFA for the root user, then stop using root.
   - Enable IAM Identity Center. Create a user, a group (`Admins`), and an `AdministratorAccess` permission set.
   - Assign the group + permission set to your account (Identity Center → AWS accounts → Assign users or groups).
   - Create a monthly budget alarm.

3. **CLI sign-in**
   ```bash
   aws configure sso          # enter your access portal URL, choose the account and role
   aws sts get-caller-identity --profile <your-profile>
   ```

4. **CDK**
   ```bash
   npm install -g aws-cdk
   cdk bootstrap aws://<account-id>/us-east-1 --profile <your-profile>
   ```

5. **Flashcards**
   - `Flashcard` component: takes `question` and `answer` as props and keeps its own `flipped` state.
   - `App`: keeps an array of cards in state, renders them with `.map()`, and has a form to add new ones.

## Try it
- `npm run dev` in `web/`, open the local URL, click a card to flip it, add a new card.
- `aws sts get-caller-identity` returns your account and an `AWSReservedSSO_AdministratorAccess` role.
- The **CDKToolkit** stack exists in CloudFormation (us-east-1).

## Check yourself
1. Add a card, then refresh the page. Why has it disappeared?
2. Why does each card flip independently, instead of all of them flipping together?
3. Why is `setFlipped((prev) => !prev)` safer than `setFlipped(!flipped)`?

<details>
<summary>Answers</summary>

1. State lives in memory. A refresh reloads the app from scratch, so the array starts again from the hard-coded cards. (Lesson 02 fixes this with `localStorage`.)
2. Each `Flashcard` has its own `useState`, so each card holds its own `flipped` value. If `flipped` lived in `App`, every card would share it.
3. The function form receives the latest state. React batches updates, so `flipped` inside a handler can be an out-of-date snapshot.

</details>

## Gotchas
- **"No AWS accounts are available to you"** during `aws configure sso`: the user exists but hasn't been assigned to the account. Create the assignment (group + permission set + account), wait a minute, run `rm -rf ~/.aws/sso/cache`, and try again.
- **`InvalidClientTokenId`**: the CLI fell back to old, invalid access keys in the `[default]` profile. Remove them from `~/.aws/credentials`, or always pass `--profile`.
- **"Unable to locate credentials" for a profile that exists**: the profile has no SSO settings (for example, it was created with `aws configure set region ...`). Use the profile that `aws configure sso` actually created, or copy its settings across in `~/.aws/config`.
- **CDK says "no credentials have been configured"** even though the CLI works: export temporary credentials into the shell, then run CDK as a separate command:
  ```bash
  eval "$(aws configure export-credentials --format env)"
  cdk bootstrap aws://<account-id>/us-east-1
  ```
  Don't put both on one line — `eval` would swallow the `cdk` command as part of the exports.
- **IAM vs IAM Identity Center**: they're different services. IAM is global (the console shows "Global"); Identity Center lives in one region.

## Further reading
- [IAM Identity Center user guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [AWS CDK developer guide](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
- [React docs: State — a component's memory](https://react.dev/learn/state-a-components-memory)
- [Vite guide](https://vite.dev/guide/)
