**🔧 CI/CD for SDET/QA — Complete Guide
-----------------------------------------------------------------------------------------------------------------------------------------
Track 1 — Jenkins Fundamentals
Q1. What is Jenkins and how does it work?
Jenkins is an open-source automation server used to build, test, and deploy code. It runs jobs triggered by events (push, schedule, webhook) and executes steps on agents.
Q2. Master vs Agent architecture
The master orchestrates pipelines and assigns work; agents are the worker nodes that execute the build steps. This allows distributed, parallel execution.
Q3. Freestyle vs Pipeline jobs
Freestyle jobs are GUI-configured, simple, limited. Pipeline jobs use a Jenkinsfile (code-as-config), support stages, conditions, loops, and are version-controlled.
Q4. Jenkins plugins ecosystem
Plugins extend Jenkins for Git, Docker, SonarQube, Slack, Allure, etc. Managed via Plugin Manager. Key for SDET: Maven, Selenium, TestNG, Allure plugins.
Q5. Build triggers
SCM polling, GitHub webhooks, timer (cron H/15 * * * *), upstream job trigger, or manual Build Now.
Q6. Build artifacts & archiving
Use archiveArtifacts to store test reports, JARs, logs. Combined with junit step for publishing test results.
Q7. Environment variables
Built-ins: BUILD_NUMBER, JOB_NAME, WORKSPACE. Custom via environment {} block or injected from credentials store.
Q8. Credentials & secrets management
Store in Jenkins Credentials Manager; access via withCredentials([]) block. Never hardcode in Jenkinsfile.
Q9. Build history & log analysis
Console output logs each step. Failed builds show exact error lines. Use currentBuild.result to check status programmatically.
Q10. Jenkins vs GitHub Actions
Jenkins: self-hosted, highly customizable, plugin-rich, complex to maintain. GitHub Actions: cloud-native, zero-infra, YAML-based, tightly integrated with GitHub.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Track 2 — Pipeline & Scripting
Q1. Declarative vs Scripted pipeline
Declarative: structured, opinionated, uses pipeline {} block — easier to read. Scripted: Groovy-based, fully flexible but verbose. 90% of modern pipelines use Declarative.
Q2. Stages, steps & post blocks
stages defines the workflow phases. steps are the commands inside each stage. post { always{} success{} failure{} } runs cleanup/notification logic.
Q3. Parallel execution
groovyparallel {
  stage('Chrome') { steps { sh 'mvn test -Dbrowser=chrome' } }
  stage('Firefox') { steps { sh 'mvn test -Dbrowser=firefox' } }
}
Reduces total pipeline time significantly.
Q4. Shared libraries
Reusable Groovy functions stored in a separate Git repo. Loaded via @Library('my-lib') _ — promotes DRY pipelines across teams.
Q5. Jenkinsfile best practices
Keep Jenkinsfile thin; logic in shared libs. Use environment {} for vars. Validate with Blue Ocean or Replay. Lint with jenkins-cli declarative-linter.
Q6. when conditions & input steps
when { branch 'main' } controls conditional stage execution. input pauses the pipeline for a human approval before proceeding to deploy.
Q7. stash / unstash
Transfer files between stages running on different agents. stash name: 'reports', includes: 'target/' → unstash 'reports' in a later stage.
Q8. Matrix builds & parameterization
Matrix allows testing across multiple dimensions (OS × browser × version) in one pipeline definition. Parameters let users pass values at runtime.
Q9. Pipeline replay & debugging
Blue Ocean has a visual debugger. Replay in classic UI lets you re-run with modified Jenkinsfile without committing. Use echo and sh 'env' to print state.
Q10. Blue Ocean vs Classic UI
Blue Ocean offers visual pipeline view, branch/PR visualization. Classic is more feature-complete for admin tasks. Blue Ocean is read-only for most configs**

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Track 3 — CD & Deployment Tools
Q1. CI vs CD (Delivery) vs CD (Deployment)
CI: automate build + test. Continuous Delivery: code is always release-ready, manual deploy trigger. Continuous Deployment: fully automated, code goes to prod on every merge.
Q2. ArgoCD & GitOps
ArgoCD is a declarative CD tool for Kubernetes. GitOps = Git is the single source of truth. ArgoCD continuously syncs the cluster state to match the Git repo.
Q3. Blue/Green vs Canary
Blue/Green: two identical environments; switch traffic instantly, easy rollback. Canary: gradually shift traffic (5% → 25% → 100%) to detect issues with partial exposure.
Q4. Rolling deployments
Replaces old pods incrementally with new ones — no downtime, but both versions run briefly. Kubernetes default deployment strategy.
Q5. Docker in CI/CD
Build Docker image in Jenkins → push to registry (ECR/DockerHub) → deploy image tag to environment. Ensures build once, deploy anywhere consistency.
Q6. Kubernetes deployments via Jenkins
Jenkins → kubectl apply -f deployment.yaml or via Helm. Use kubectl plugin or service account credentials. Namespaces separate environments (dev/staging/prod).
Q7. Environment promotion strategy
Code flows: Dev → QA → Staging → Prod. Each promotion can require gate checks: test pass %, security scan, manual approval.
Q8. Rollback strategies
Kubernetes: kubectl rollout undo deployment/myapp. ArgoCD: sync to previous Git commit. Blue/Green: switch LB back to old environment.
Q9. Release gates & approval flows
Use Jenkins input step or ArgoCD sync windows. Automated gates: SonarQube quality gate, test coverage threshold, DAST scan results.
Q10. Helm charts & templating
Helm packages Kubernetes manifests as charts. values.yaml allows environment-specific overrides. helm upgrade --install is the CD command used in pipelines.**

----------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Track 4 — QA Integration in CI/CD
Q1. Test stages in a CI pipeline
Typical order: Unit Tests → Static Analysis → Build → Integration Tests → API Tests → Smoke Tests → Deploy → E2E Tests.
Q2. Fail-fast vs Full suite
Fail-fast stops the pipeline on first failure (saves time in CI). Full suite runs everything and collects all failures (useful for nightly regression runs).
Q3. Test reports: JUnit XML & Allure
Jenkins junit 'target/surefire-reports/*.xml' publishes results natively. Allure plugin generates rich visual reports with history trends and categories.
Q4. Flaky tests in CI
Strategies: mark as known flaky and quarantine, retry mechanism (@Retry(3)), analyze root cause (timing, test isolation). Never ignore flakes — they erode trust in CI.
Q5. Smoke vs Regression in pipeline
Smoke: small, critical-path tests run after every deploy (fast, ~5 min). Regression: full suite run on schedule or pre-release (comprehensive, may take hours).
Q6. Code coverage gates
Jacoco generates coverage reports. SonarQube Quality Gate can block the pipeline if coverage drops below threshold (e.g., 80%). Configured in SonarQube project settings.
Q7. Selenium Grid in Jenkins pipeline
Spin up Selenium Grid via Docker Compose in pipeline. Run tests pointing to Grid hub URL. Tear down after. Alternatively use Selenoid or BrowserStack.
Q8. API test stage with RestAssured
Add a Maven/Gradle stage that runs RestAssured tests. Publish results as JUnit XML. Can run in parallel with UI tests to save time.
Q9. Notifications: Slack / email on fail
groovypost {
  failure {
    slackSend channel: '#qa-alerts', message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
  } **
}
Q10. Test environment provisioning
Use Docker Compose or Terraform to spin up environment before tests. Tear down after. Ensures clean, isolated, reproducible test environments per build.**
