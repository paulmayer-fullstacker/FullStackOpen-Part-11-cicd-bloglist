# Full Stack Open: FullStackOpen-Part-11-cicd-bloglist

## Introduction:

The FullStackOpen-Part-11-cicd-bloglist repository implements a complete Continuous Integration and Continuous Deployment (CI/CD) pipeline for the full-stack Bloglist application using GitHub Actions. The pipeline automates code quality checks, building, unit and integration testing, and end-to-end (E2E) browser testing on every push and pull request.

Key components of the solution include:

- Automated CI Workflow: Configured via GitHub Actions to automatically run ESLint for linting, execute backend unit/integration tests with Jest, and run frontend component tests.

- Hermetic E2E Testing with Playwright: Spins up a dedicated test backend running against a test MongoDB database, ensuring clean database resets between tests for full isolation.

- Full-Stack End-to-End Coverage: Tests complete user flows including user creation, login verification, blog creation, liking, creator-restricted deletion, and dynamic re-sorting by likes.

- Robust UI Synchronization: Solves asynchronous race conditions between the Playwright runner and React DOM updates, guaranteeing stable test execution across both local and CI environments.

Herewith brief notes on running and testing the FullStackOpen-Part-11-cicd-bloglist solution.

---

## The Local Installation

From GitHub, take a copy of the clonable repository git URL. Then, open the terminal, navigate to the directory where you want to store the project, and run:

```bash
git clone <THE_REPOSITORY_GIT_URL>
cd FullStackOpen-Part-11-cicd-bloglist
```

Run the root script (from the root repo directory) to automatically install dependencies across the backend, frontend, and E2E test directories in one command:

```bash
npm run install:all
```

Install Playwright Browsers. Run the dedicated script to download the necessary Playwright browser binaries and system dependencies:

```bash
npm run install:playwright
```

## Local testing

### Prerequisites

Ensure that our backend .env (bloglist-backend-part4/.env) has a valid TEST_MONGODB_URI set.

### Step-by-Step Test Procedure

#### Step-1: Run Backend Unit/Integration Tests

From the root repo directory, navigate to the backend directory and run the Jest/Node test suite:

```bash
cd bloglist-backend-part4
npm run test
cd ..
```

#### Step-2: Run Frontend Component Tests (React / Vitest / Jest)

Navigate to your frontend project directory to run component/unit tests:

```bash
cd bloglist-frontend-main
npm run test
cd ..
```

#### Step-3: Run Full E2E Playwright Tests

To run E2E tests, start the backend server in test mode on port 3003:

```bash
cd bloglist-backend-part4
npm run start:test
```

Wait for it to accept connections, and launch Playwright. Then, from the root repo directory:

```bash
npm run test:e2e
```

Remember to shutdown the backend server (<ctrl-C>), when testing is complete.

## Automated Continuous Integration Testing

On pushing new/amended code to the repo, our GitHub Actions workflow executes a Continuous Integration (CI) pipeline (based on our `.github/workflows/pipeline.yml` file). The Continuous Integration Test Suite combines all (front, back and e2e) tests. To trigger the workflow, make a simple modification to this ReadMe file and save. Then commit and push the new code:

```bash
git add .
git commit -m "Workflow trigger"
git push origin main
```

## Challenges

### Concurrency Issues with Rapid Form/Button Interactions:

Rapidly looping over the like button triggered concurrent HTTP PUT requests. Fast DOM re-renders detached the button element mid-loop, causing Playwright to drop subsequent clicks and causing count assertions (e.g., likes 5) to time out.

### Solution:

UI Synchronization in `tests/helper.js`: Updated `likeBlogMultiTimes` to use a 1-based loop index `(i = 1; i <= n; i++)` that explicitly awaits UI state changes (`await blogEntry.getByText(likes ${i}).waitFor()`) after every click, eliminating race conditions.

### Stale DOM Locators During Re-sorting:

Re-sorting blogs by likes after each click caused bound locator handles (blogEntry) to point to stale elements, breaking subsequent interactions and position checks.

### Solution:

Dynamic Locators in `bloglist_app.spec.js`: Implemented a dynamic locator getter function (`getBlogEntry = (title) => page.locator(...)`) to fetch fresh DOM elements dynamically after state re-renders.

Explicit Order Verification: Asserted both exact array matches (`allTextContents()`) and explicit position indexes (`.nth(0), .nth(1)`) to guarantee correct descending order.

---

## Exercise 22

Switched to new development platform. So, made test commits to confirm baseline.

<!-- Pipeline test on push to main -->

<!-- Pipeline test on feature branch merge to main -->

---

<br/>

<hr style="height: 5px; background-color: black; border: none;">
