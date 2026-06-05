# 🚀 CI/CD Interview Prep for SDET & QA Engineers

> **40 Questions · 4 Tracks · Jenkins + ArgoCD / Docker / Kubernetes**  
> Curated from real interview rounds — with answers, examples & code snippets

---

## 📋 Table of Contents

- [Track 1 — Jenkins Fundamentals](#track-1--jenkins-fundamentals)
- [Track 2 — Pipeline & Scripting](#track-2--pipeline--scripting)
- [Track 3 — CD & Deployment Tools](#track-3--cd--deployment-tools)
- [Track 4 — QA Integration in CI/CD](#track-4--qa-integration-in-cicd)
- [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## Track 1 — Jenkins Fundamentals

### Q1. What is Jenkins and how does it work?

Jenkins is an open-source automation server used to **build, test, and deploy** code automatically.

**How it works:**
- A trigger fires (code push, webhook, schedule)
- Jenkins picks up the job and runs it on an agent
- Steps execute sequentially (or in parallel)
- Results (pass/fail, reports, artifacts) are stored

```
Developer pushes code
        ↓
  GitHub Webhook fires
        ↓
  Jenkins picks up job
        ↓
  Build → Test → Deploy
        ↓
  Notify (Slack/Email)
```

---

### Q2. Master vs Agent Architecture

| Component | Role |
|-----------|------|
| **Master** | Orchestrates jobs, stores config, schedules builds |
| **Agent** | Worker node that actually executes the build steps |

**Why it matters for QA:**  
You can run tests in parallel across multiple agents — e.g., Chrome agent, Firefox agent, API agent — all triggered from one pipeline.

```groovy
pipeline {
  agent { label 'selenium-chrome' }  // runs on a specific agent
}
```

---

### Q3. Freestyle vs Pipeline Jobs

| Feature | Freestyle | Pipeline |
|---------|-----------|----------|
| Configuration | GUI (click-based) | Code (`Jenkinsfile`) |
| Version control | ❌ No | ✅ Yes |
| Parallel stages | ❌ Limited | ✅ Native support |
| Conditions/loops | ❌ No | ✅ Full Groovy |
| Recommended for | Quick one-off jobs | All production use |

> ✅ **Always prefer Pipeline jobs** — they are code-as-config, version-controlled, and repeatable.

---

### Q4. Jenkins Plugins Ecosystem

Jenkins is extended through plugins. Key plugins for SDETs:

| Plugin | Purpose |
|--------|---------|
| `Git` | Source code integration |
| `Maven Integration` | Build Java/Maven projects |
| `TestNG Results` | Publish TestNG reports |
| `Allure` | Rich visual test reports |
| `SonarQube Scanner` | Code quality gates |
| `Slack Notification` | Notify team on pass/fail |
| `Docker Pipeline` | Build & run Docker images |
| `Kubernetes` | Run agents on K8s pods |

---

### Q5. Build Triggers

Ways to start a Jenkins build:

```groovy
triggers {
  // Poll SCM every 5 minutes
  pollSCM('H/5 * * * *')

  // Scheduled build (nightly regression at 2 AM)
  cron('0 2 * * *')

  // GitHub webhook (recommended — real-time)
  githubPush()
}
```

> 💡 **Interview tip:** Prefer webhooks over polling — polling wastes resources and has delay.

---

### Q6. Build Artifacts & Archiving

```groovy
post {
  always {
    // Archive test reports
    junit 'target/surefire-reports/*.xml'

    // Archive files for download
    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

    // Publish Allure report
    allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
  }
}
```

---

### Q7. Environment Variables in Jenkins

**Built-in variables:**

| Variable | Value |
|----------|-------|
| `BUILD_NUMBER` | Current build number |
| `JOB_NAME` | Name of the job |
| `WORKSPACE` | Build workspace path |
| `GIT_BRANCH` | Branch being built |
| `BUILD_URL` | Full URL to this build |

**Custom variables in Jenkinsfile:**

```groovy
environment {
  APP_ENV     = 'staging'
  BASE_URL    = 'https://staging.myapp.com'
  BROWSER     = 'chrome'
}
```

---

### Q8. Credentials & Secrets Management

> ⚠️ **Never hardcode passwords or API keys in Jenkinsfile.**

Store secrets in **Jenkins Credentials Manager**, then access them:

```groovy
environment {
  DB_PASSWORD = credentials('db-password-id')
  API_KEY     = credentials('api-key-id')
}

// Or inside a step
steps {
  withCredentials([usernamePassword(
    credentialsId: 'app-credentials',
    usernameVariable: 'USER',
    passwordVariable: 'PASS'
  )]) {
    sh 'curl -u $USER:$PASS https://api.example.com'
  }
}
```

---

### Q9. Build History & Log Analysis

- **Console Output** — full real-time log of every step
- `currentBuild.result` — `SUCCESS`, `FAILURE`, `UNSTABLE`, `ABORTED`
- **Build History** in sidebar shows trend (pass/fail streak)
- Use `catchError` to continue pipeline after a failure and still capture logs

```groovy
steps {
  catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
    sh 'mvn test'
  }
}
```

---

### Q10. Jenkins vs GitHub Actions

| Feature | Jenkins | GitHub Actions |
|---------|---------|----------------|
| Hosting | Self-hosted | Cloud-native (GitHub) |
| Setup | Complex | Zero-infra |
| Config | Jenkinsfile (Groovy) | YAML workflows |
| Plugin ecosystem | 1800+ plugins | Marketplace actions |
| Cost | Free (infra cost) | Free tier + paid |
| Best for | Enterprise, complex | Open source, GitHub repos |

---

## Track 2 — Pipeline & Scripting

### Q1. Declarative vs Scripted Pipeline

```groovy
// ✅ DECLARATIVE — Structured, readable, recommended
pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
  }
}

// ⚠️ SCRIPTED — Full Groovy, flexible but verbose
node {
  stage('Test') {
    sh 'mvn test'
  }
}
```

> Use **Declarative** for 90% of cases. Use Scripted only when you need advanced Groovy logic.

---

### Q2. Stages, Steps & Post Blocks

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
  }

  post {
    always   { junit 'target/surefire-reports/*.xml' }
    success  { echo '✅ Build passed!' }
    failure  { slackSend message: '❌ Build failed!' }
    unstable { echo '⚠️ Tests unstable (some failures)' }
  }
}
```

---

### Q3. Parallel Execution in Pipeline

Dramatically reduces pipeline time by running stages simultaneously:

```groovy
stage('Run Tests in Parallel') {
  parallel {
    stage('Chrome Tests') {
      steps { sh 'mvn test -Dbrowser=chrome' }
    }
    stage('Firefox Tests') {
      steps { sh 'mvn test -Dbrowser=firefox' }
    }
    stage('API Tests') {
      steps { sh 'mvn test -Dtest=ApiTestSuite' }
    }
  }
}
```

> ⏱️ 3 suites that each take 10 mins → parallel runs in ~10 mins instead of 30.

---

### Q4. Shared Libraries in Jenkins

Reusable Groovy code stored in a separate Git repo — DRY across all pipelines.

**Structure of shared library repo:**
```
vars/
  sendSlackAlert.groovy
  runSeleniumTests.groovy
src/
  org/myteam/TestUtils.groovy
```

**Using in Jenkinsfile:**
```groovy
@Library('my-shared-library') _

pipeline {
  stages {
    stage('Test') {
      steps {
        runSeleniumTests(browser: 'chrome', env: 'staging')
      }
    }
    post {
      failure { sendSlackAlert('#qa-alerts', 'Tests failed!') }
    }
  }
}
```

---

### Q5. Jenkinsfile Best Practices

```groovy
// ✅ DO: Use environment block for variables
environment {
  TEST_ENV = 'staging'
}

// ✅ DO: Use credentials() for secrets
environment {
  API_KEY = credentials('api-key')
}

// ✅ DO: Add timeout to prevent hung builds
options {
  timeout(time: 30, unit: 'MINUTES')
  disableConcurrentBuilds()
}

// ❌ DON'T: Hardcode values
sh 'curl -H "Auth: mypassword123" ...'

// ❌ DON'T: Put heavy logic in Jenkinsfile — move to shared lib
```

---

### Q6. `when` Conditions & `input` Steps

```groovy
// Run stage only on main branch
stage('Deploy to Prod') {
  when {
    branch 'main'
    // or: expression { return params.DEPLOY_TO_PROD == true }
  }
  steps {
    // Manual approval gate
    input message: 'Deploy to Production?', ok: 'Yes, Deploy!'
    sh './deploy.sh prod'
  }
}
```

---

### Q7. stash / unstash Between Stages

Transfer files between stages (especially useful when stages run on different agents):

```groovy
stage('Build') {
  steps {
    sh 'mvn package'
    stash name: 'built-app', includes: 'target/*.jar'
  }
}

stage('Test') {
  agent { label 'selenium-agent' }
  steps {
    unstash 'built-app'   // get the JAR from Build stage
    sh 'mvn test'
    stash name: 'test-reports', includes: 'target/surefire-reports/**'
  }
}
```

---

### Q8. Matrix Builds & Parameterization

```groovy
// Matrix: test across multiple OS and browsers
matrix {
  axes {
    axis { name 'BROWSER'; values 'chrome', 'firefox', 'edge' }
    axis { name 'OS';      values 'linux', 'windows' }
  }
  stages {
    stage('Cross-browser Tests') {
      steps {
        sh "mvn test -Dbrowser=${BROWSER} -Dos=${OS}"
      }
    }
  }
}

// Parameterized build
parameters {
  string(name: 'TARGET_ENV', defaultValue: 'staging', description: 'Environment to test')
  booleanParam(name: 'RUN_REGRESSION', defaultValue: false)
  choice(name: 'BROWSER', choices: ['chrome', 'firefox', 'edge'])
}
```

---

### Q9. Pipeline Replay & Debugging

| Tool | How to use |
|------|-----------|
| **Replay** | Re-run a build with modified Jenkinsfile (no commit needed) |
| **Blue Ocean** | Visual pipeline view, click into failed step |
| `echo` | Print variable values to console |
| `sh 'env'` | Print all environment variables |
| `catchError` | Continue pipeline after failure, collect logs |

---

### Q10. Blue Ocean vs Classic UI

| Feature | Blue Ocean | Classic UI |
|---------|-----------|------------|
| Pipeline visualization | ✅ Visual graph | ❌ Text only |
| Branch/PR view | ✅ Built-in | ❌ Manual setup |
| Admin config | ❌ Limited | ✅ Full access |
| Plugin management | ❌ No | ✅ Yes |
| Recommended for | Developers viewing builds | Admins configuring Jenkins |

---

## Track 3 — CD & Deployment Tools

### Q1. CI vs CD (Delivery) vs CD (Deployment)

```
Continuous Integration (CI)
  → Auto build + test on every commit
  → Goal: catch bugs early

Continuous Delivery (CD - Delivery)
  → Code is ALWAYS in a deployable state
  → Deploy is manual (a button click)
  → Goal: be ready to release anytime

Continuous Deployment (CD - Deployment)
  → Every passing commit auto-deploys to PRODUCTION
  → Zero manual steps
  → Goal: maximum release velocity
```

> 💡 **Interview tip:** Most companies do Continuous Delivery, not full Deployment — the distinction is the manual approval gate.

---

### Q2. ArgoCD & GitOps Principles

**GitOps = Git is the single source of truth for infrastructure state.**

```
Developer commits Kubernetes manifest → Git repo
ArgoCD detects drift between Git state and cluster state
ArgoCD automatically syncs cluster to match Git
```

**Key ArgoCD concepts:**

| Concept | Meaning |
|---------|---------|
| Application | A set of K8s resources to be deployed |
| Sync | Making cluster match Git state |
| Health | Is the deployed app running correctly? |
| Rollback | Revert by pointing to previous Git commit |

---

### Q3. Blue/Green vs Canary Deployments

**Blue/Green:**
```
BLUE (current - 100% traffic)  ←── users
GREEN (new version - 0% traffic) ←── testing

  Switch LB → GREEN gets 100% traffic
  Keep BLUE for instant rollback
```

**Canary:**
```
v1 (old) ←── 90% traffic
v2 (new) ←──  10% traffic  ← monitor errors/latency

  Gradually shift: 10% → 25% → 50% → 100%
  Roll back if error rate spikes
```

| Strategy | Rollback Speed | Risk | Infra Cost |
|----------|---------------|------|-----------|
| Blue/Green | Instant | Low | 2× servers |
| Canary | Fast | Very Low | Small extra |
| Rolling | Slower | Medium | Normal |

---

### Q4. Rolling Deployments

Kubernetes default strategy — replaces pods gradually:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # create 1 extra pod before terminating old
    maxUnavailable: 0  # never have 0 available pods
```

```
Pod v1, Pod v1, Pod v1, Pod v1
           ↓ rolling update
Pod v2, Pod v1, Pod v1, Pod v1
Pod v2, Pod v2, Pod v1, Pod v1
Pod v2, Pod v2, Pod v2, Pod v1
Pod v2, Pod v2, Pod v2, Pod v2  ✅ Done
```

---

### Q5. Docker in CI/CD Pipelines

```groovy
pipeline {
  stages {
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t myapp:${BUILD_NUMBER} .'
      }
    }
    stage('Push to Registry') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub', ...)]) {
          sh 'docker push myrepo/myapp:${BUILD_NUMBER}'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh "kubectl set image deployment/myapp myapp=myrepo/myapp:${BUILD_NUMBER}"
      }
    }
  }
}
```

> Build once → tag with `BUILD_NUMBER` → deploy same image across all environments.

---

### Q6. Kubernetes Deployments via Jenkins

```groovy
stage('Deploy to Kubernetes') {
  steps {
    withKubeConfig([credentialsId: 'k8s-service-account']) {
      // Apply manifest
      sh 'kubectl apply -f k8s/deployment.yaml -n staging'

      // Or via Helm
      sh """
        helm upgrade --install myapp ./helm/myapp \
          --namespace staging \
          --set image.tag=${BUILD_NUMBER} \
          --set env=staging
      """

      // Wait for rollout to complete
      sh 'kubectl rollout status deployment/myapp -n staging'
    }
  }
}
```

---

### Q7. Environment Promotion Strategy

```
Dev  → QA  → Staging  → Production

Each promotion requires:
  ✅ Previous environment tests passed
  ✅ Code coverage ≥ 80%
  ✅ SonarQube Quality Gate: PASSED
  ✅ Security scan: no critical vulnerabilities
  ✅ Manual approval (for Staging → Prod)
```

Implemented in Jenkins using `when` conditions + `input` approval gates between stages.

---

### Q8. Rollback Strategies in Production

| Tool | Rollback Command |
|------|-----------------|
| **Kubernetes** | `kubectl rollout undo deployment/myapp` |
| **Helm** | `helm rollback myapp 2` (rollback to revision 2) |
| **ArgoCD** | Click "Rollback" in UI or sync to previous Git commit |
| **Blue/Green** | Switch load balancer back to Blue environment |

```bash
# Check rollout history
kubectl rollout history deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=3

# Verify rollback
kubectl rollout status deployment/myapp
```

---

### Q9. Release Gates & Approval Flows

**Automated gates** (pipeline fails if not met):
- Test pass rate ≥ 95%
- Code coverage ≥ 80% (Jacoco/SonarQube)
- No critical security vulnerabilities (OWASP ZAP / Trivy)
- Performance baseline within acceptable range

**Manual approval gate** (Jenkins `input` step):

```groovy
stage('Approval for Production') {
  steps {
    input(
      message: 'Approve deployment to Production?',
      submitter: 'qa-lead,release-manager',
      ok: 'Deploy to Prod'
    )
  }
}
```

---

### Q10. Helm Charts & Templating Basics

Helm packages Kubernetes manifests as reusable charts.

**Structure:**
```
myapp/
  Chart.yaml          # chart metadata
  values.yaml         # default configuration
  values-staging.yaml # staging overrides
  values-prod.yaml    # prod overrides
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
```

**Template example:**
```yaml
# templates/deployment.yaml
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
replicas: {{ .Values.replicaCount }}
```

**Deploy command in Jenkins:**
```bash
helm upgrade --install myapp ./myapp \
  -f values.yaml \
  -f values-${TARGET_ENV}.yaml \
  --set image.tag=${BUILD_NUMBER}
```

---

## Track 4 — QA Integration in CI/CD

### Q1. Test Stages in a CI Pipeline

```
Commit → Build → Unit Tests → Static Analysis
         → Integration Tests → API Tests
         → Deploy to Test Env → Smoke Tests
         → E2E / Regression Tests → Report
```

**Typical time budgets:**

| Stage | Target Time |
|-------|------------|
| Unit tests | < 5 min |
| API tests | < 10 min |
| Smoke tests | < 5 min |
| Full E2E regression | 30–60 min (nightly) |

---

### Q2. Fail-Fast vs Full Suite Strategy

| Strategy | When to use | Behaviour |
|----------|-------------|-----------|
| **Fail-fast** | CI on every PR | Stop on first failure, save time |
| **Full suite** | Nightly regression | Run everything, collect all failures |

```groovy
// Fail-fast (default Maven behaviour)
sh 'mvn test'

// Full suite — collect all results even on failure
sh 'mvn test || true'
post {
  always { junit 'target/surefire-reports/*.xml' }
}
```

---

### Q3. Test Reports — JUnit XML & Allure

```groovy
post {
  always {
    // Native Jenkins JUnit report
    junit 'target/surefire-reports/*.xml'

    // Allure rich report (requires Allure plugin)
    allure([
      includeProperties: false,
      results: [[path: 'allure-results']]
    ])
  }
}
```

**Allure advantages over plain JUnit:**
- 📊 Test trend history across builds
- 🏷️ Categories (product bugs, test bugs, broken tests)
- 📸 Screenshots & logs attached to failures
- 🔍 Filterable by feature, story, severity

---

### Q4. Flaky Tests in CI — Handling Them

**What is a flaky test?**  
A test that passes and fails intermittently without code changes.

**Handling strategies:**

```groovy
// Option 1: Retry in TestNG
@Test(retryAnalyzer = RetryAnalyzer.class)
public void testCheckout() { ... }

// Option 2: Retry in Maven Surefire
<plugin>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration>
    <rerunFailingTestsCount>2</rerunFailingTestsCount>
  </configuration>
</plugin>

// Option 3: Mark as UNSTABLE (not FAILED) in Jenkins
post {
  unstable { echo 'Some tests flaky — investigate!' }
}
```

> ⚠️ Never ignore flaky tests. They erode confidence in CI. Quarantine → fix → re-enable.

---

### Q5. Smoke vs Regression in Pipeline

```groovy
stage('Post-Deploy Smoke Tests') {
  // Runs after every deployment — fast, critical paths only
  steps {
    sh 'mvn test -Dgroups=smoke -Denv=${TARGET_ENV}'
    // ~5-10 min, covers login, homepage, key user journeys
  }
}

stage('Nightly Regression') {
  when {
    cron('0 1 * * *')  // 1 AM nightly
  }
  steps {
    sh 'mvn test -Dgroups=regression'
    // Full suite, all scenarios, 30-60 min
  }
}
```

---

### Q6. Code Coverage Gates — Jacoco & SonarQube

```groovy
stage('Code Quality Gate') {
  steps {
    withSonarQubeEnv('SonarQube') {
      sh 'mvn sonar:sonar -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml'
    }
  }
}

stage('Quality Gate Check') {
  steps {
    timeout(time: 5, unit: 'MINUTES') {
      waitForQualityGate abortPipeline: true
      // Pipeline fails if coverage < threshold or bugs/vulnerabilities found
    }
  }
}
```

SonarQube Quality Gate checks: code coverage %, duplications, bugs, vulnerabilities, code smells.

---

### Q7. Selenium Grid in Jenkins Pipeline

```groovy
stage('Start Selenium Grid') {
  steps {
    sh 'docker-compose -f selenium-grid.yml up -d'
    sh 'sleep 10'  // wait for grid to be ready
  }
}

stage('Run Selenium Tests') {
  steps {
    sh """
      mvn test \
        -Dbrowser=chrome \
        -DgridUrl=http://localhost:4444/wd/hub \
        -Denv=staging
    """
  }
}

stage('Teardown Grid') {
  steps {
    sh 'docker-compose -f selenium-grid.yml down'
  }
}
```

> Alternatives: **Selenoid** (faster Docker-based grid), **BrowserStack** / **Sauce Labs** (cloud).

---

### Q8. API Test Stage with RestAssured

```groovy
stage('API Tests') {
  steps {
    sh """
      mvn test \
        -Dtest=ApiTestSuite \
        -Dbase.url=${BASE_URL} \
        -Dapi.key=${API_KEY}
    """
  }
  post {
    always {
      junit 'target/api-test-reports/*.xml'
    }
  }
}
```

Run API tests **before** UI tests — they're faster and catch backend issues early without spinning up a browser.

---

### Q9. Notifications — Slack & Email on Fail

```groovy
post {
  failure {
    // Slack notification
    slackSend(
      channel: '#qa-alerts',
      color: 'danger',
      message: """
        ❌ *BUILD FAILED*
        Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
        Branch: ${env.GIT_BRANCH}
        URL: ${env.BUILD_URL}
      """
    )

    // Email notification
    emailext(
      subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
      body: "Check console: ${env.BUILD_URL}console",
      to: 'qa-team@company.com'
    )
  }

  success {
    slackSend(channel: '#qa-alerts', color: 'good',
              message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} passed!")
  }
}
```

---

### Q10. Test Environment Provisioning

**Problem:** Inconsistent environments cause flaky tests.  
**Solution:** Spin up a fresh, isolated environment for every pipeline run.

```groovy
stage('Provision Test Environment') {
  steps {
    // Docker Compose approach
    sh 'docker-compose -f docker-compose.test.yml up -d'
    sh './scripts/wait-for-app.sh http://localhost:8080/health'
  }
}

stage('Run Tests') {
  steps {
    sh 'mvn test -Dbase.url=http://localhost:8080'
  }
}

stage('Teardown') {
  steps {
    sh 'docker-compose -f docker-compose.test.yml down -v'
  }
}
```

> Each build gets a **clean database, fresh app state, isolated network** — no test pollution.

---

## Quick Reference Cheatsheet

### Pipeline Structure Skeleton

```groovy
pipeline {
  agent any

  options {
    timeout(time: 30, unit: 'MINUTES')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  environment {
    APP_ENV  = 'staging'
    BASE_URL = 'https://staging.myapp.com'
    API_KEY  = credentials('api-key-id')
  }

  parameters {
    choice(name: 'BROWSER', choices: ['chrome', 'firefox'], description: 'Browser to test')
    booleanParam(name: 'RUN_REGRESSION', defaultValue: false)
  }

  triggers {
    cron('0 2 * * *')        // nightly at 2 AM
    pollSCM('H/5 * * * *')   // or poll SCM
  }

  stages {
    stage('Build')            { steps { sh 'mvn clean package -DskipTests' } }
    stage('Unit Tests')       { steps { sh 'mvn test -Dgroups=unit' } }
    stage('API Tests')        { steps { sh 'mvn test -Dgroups=api' } }
    stage('Deploy to Staging'){ steps { sh './deploy.sh staging' } }
    stage('Smoke Tests')      { steps { sh 'mvn test -Dgroups=smoke' } }
    stage('Deploy to Prod') {
      when { branch 'main' }
      steps {
        input 'Approve production deployment?'
        sh './deploy.sh prod'
      }
    }
  }

  post {
    always   { junit 'target/surefire-reports/*.xml' }
    success  { slackSend color: 'good',   message: "✅ ${JOB_NAME} passed" }
    failure  { slackSend color: 'danger', message: "❌ ${JOB_NAME} failed" }
    unstable { slackSend color: 'warning',message: "⚠️ ${JOB_NAME} unstable" }
  }
}
```

---

### Deployment Strategy Decision Guide

```
Is zero-downtime critical?
  └─ YES → Blue/Green or Canary
  └─ NO  → Rolling (simpler, less infra)

Is gradual rollout important (catch production issues early)?
  └─ YES → Canary
  └─ NO  → Blue/Green (simpler, instant switch)

Is GitOps / Kubernetes the platform?
  └─ YES → ArgoCD + Helm
  └─ NO  → Jenkins deploy step with kubectl/scripts
```

---

### Common Jenkins Interview Questions — 1-Line Answers

| Question | Answer |
|----------|--------|
| What is a Jenkinsfile? | Pipeline-as-code stored in source control |
| What is a Jenkins agent? | Worker node that runs build/test steps |
| What is `post { always }` for? | Runs regardless of build result (cleanup, reports) |
| What does `stash` do? | Temporarily saves files to transfer between stages/agents |
| What is a shared library? | Reusable Groovy code shared across multiple pipelines |
| What is Blue/Green deployment? | Two identical envs, switch traffic instantly for zero-downtime |
| What is a Quality Gate? | Automated check (e.g., coverage ≥ 80%) that must pass to proceed |
| What is GitOps? | Git as the single source of truth for infrastructure state |
| What is ArgoCD? | Declarative, GitOps-based CD tool for Kubernetes |
| What is a Canary release? | Gradual traffic shift to new version to reduce risk |

---

## 📚 Resources

- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)

---

## ⭐ Found this useful?

Star this repo and share with your QA network!

*by **Abhijeet Jagtap** — QA Mentor Hub*
