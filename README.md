# Capstone Track: Cybersecurity and Governance
 
**Final Sprint | Due before Final Presentations**

> **New to this track? Start here.** The shared [INET 4031 Capstone Guide](https://github.com/INET4031-Labs/inet4031-capstone-guide) explains how to work through any track on your own. Feeling lost in the first few days is normal for a capstone: read the guide's Start Here page, run the Environment Checkpoint below, and take the next small step.
>
> | I need... | Go to |
> |---|---|
> | A reading order and what to do in my first session | [START-HERE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/START-HERE.md) |
> | What I may change in the app, and how to propose my own criterion | [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) |
> | Local VM: how my k3s and OpenTofu setup differs from this README | [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) |
> | What to prioritize in Weeks 11 to 14 | [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md) and the Sprint Timeline below |
> | Plain-language explanations of this track's concepts | [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) |
> | A cheat sheet for each tool in this track | [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md) |
> | Which file proves which success criterion | [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md) |
> | How the Week 13 rebuild works | [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md) and the Rebuild section below |
> | Help when something is broken | [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) |
> | How to use AI tools well on this project | [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md) |
>
> **Local VM students:** much of this README describes the Shared VM setup (k3d, team numbers, `main`, `kubectl apply`). Your setup is a single-node k3s cluster that changes through OpenTofu and your deploy script. Look for the **Local VM** notes in each section, and keep [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) open while you work.
 
## Overview
 
Elena Brandt, Harborlight Logistics' Compliance Officer, has a problem. The contract renewal with Harborlight's largest customer is in six weeks, and that customer's external auditor wants evidence that the `statustracker` platform meets a baseline of CIS-style security controls. Today, the Platform Team (Sam Whitfield and two other engineers) can say "we did RBAC, NetworkPolicy, and Trivy scanning in CI," but nobody can hand an auditor a document that proves it. There is no list of controls, no saved scan output, no risk register, and no written policy.
 
Your team is the Platform Team's compliance helper for this sprint. You will turn the security work you did in earlier weeks into audit evidence: scan your own image, repository, manifests, and cluster; check the results against a short list of controls; fix what you can; write down what you cannot fix as an accepted risk; and package everything so a non-engineer can follow it. Budget is fixed: no new paid services, and everything must run on the existing single server.
 
After completing this track, you will have a repeatable evidence-collection script, a control matrix mapping each control to saved evidence, a risk register, three short policy documents, and a runbook that lets Elena (or an auditor) re-check the platform without asking an engineer.
 
## The Business Problem
 
**Who is asking:** Elena Brandt, Compliance Officer. Sam Whitfield (Platform Team lead) will review the technical work. Marcus Reyes (CTO) wants to know the cost, which must be zero.
 
**Pain today:** Security controls exist but are undocumented. When the auditor asks "how do you know no container runs as root?" or "who can read your Kubernetes secrets?", the answer is a screenshot someone took weeks ago. Findings from scans are not tracked, so nobody can say whether the platform is getting safer or worse.
 
**Constraints:**
- Budget: no new paid services. Everything runs on the existing single server with free, lightweight CLI tools.
- Deadline: evidence must be ready before the contract renewal in six weeks (for you: before Final Presentations).
- Scope: you scan only your own image, your own repo, and your own cluster.
- Evidence must be reproducible. An auditor will not accept a one-time screenshot.
**Success criteria** (your team proves each one with a file in your repo):
 
1. **Control matrix:** `capstone/cybersecurity-governance/controls.csv` lists at least 15 CIS-style controls (covering image, repo, Kubernetes workload, RBAC, network, and process). Every row has an ID, a description, a status of `Pass`, `Fail`, or `Accepted Risk`, and a path to the evidence file that proves the status. No row is missing evidence.
2. **Image and manifest scanning:** Saved scan reports exist for your Flask image and your Kubernetes manifests. Your team reduces the count of HIGH and CRITICAL findings in the image scan by at least one finding (or documents why none could be fixed), shown by a before report and an after report. Any remaining CRITICAL finding appears in the risk register.
3. **Secrets scanning:** A gitleaks report covering your repo's full git history exists. It shows 0 unresolved findings; every finding is either rotated/removed or marked as a documented false positive. (Real secret values never appear in your docs.)
4. **Access and network verification:** At least 8 `kubectl auth can-i` checks for the `flask-app` ServiceAccount and one other identity are saved with expected and actual results, and at least 2 tests prove the default-deny NetworkPolicy blocks traffic that should be blocked. Every result matches what you predicted in writing.
5. **Governance documents and repeatability:** `capstone/cybersecurity-governance/risk-register.md` has at least 8 risks, each with a likelihood (1-5), impact (1-5), score, owner (from the Harborlight cast), and treatment. At least 3 one-page policies exist (for example access control, vulnerability management, secrets handling). One script runs all scans and writes a timestamped evidence folder in under 5 minutes.
How you get there is up to you.
 
## Learning Objectives
 
- Translate a security framework (CIS-style controls) into concrete, checkable tests
- Run image, configuration, and secrets scanners and triage the findings into fix, accept, or false positive
- Verify Kubernetes RBAC and NetworkPolicy behavior with evidence instead of assumptions
- Write a risk register that scores, assigns, and tracks risks
- Produce audit evidence that is reproducible, dated, and understandable by a non-engineer
- Explain the difference between a control, a test of that control, and evidence of the test
## Prerequisites
 
- Week 7 complete: RBAC (`flask-app` ServiceAccount), SecurityContext (UID 1000, read-only root filesystem, drop ALL), and the NetworkPolicies (default-deny-ingress plus the nginx-to-flask, flask-to-db, ingress-to-nginx, and monitoring-to-flask allows) are applied
- Week 6 complete: GitHub Actions CI with the Trivy step, and branch protection on `main`
- Week 5 complete: Prometheus running in the `monitoring` namespace, and `week-5/pq.sh` works
- Week 2 Compose stack and Week 3 k3d cluster running with the Flask Deployment using image `week-2-flask`

> **Local VM (Ubuntu, individual):** the weeks above are Shared VM weeks. Your equivalents:
>
> - **Week 8 (security hardening):** `web` runs as UID 10001 with `run_as_non_root`, no privilege escalation, all capabilities dropped, and CPU and memory limits; `db` has limits; the Deploy gate's `security scan` job runs Checkov on `week-5/`; the `protect-master` ruleset requires all three gate jobs.
> - **Week 9:** the NetworkPolicy is now `db-clients`, which lets only `web` and `db-exporter` reach `db` on 5432. There is **no default-deny** policy and no separate ServiceAccount: `web` runs as the `default` ServiceAccount in `default`.
> - **Week 6:** Prometheus in `monitoring`, and `week-6/pq.sh`.
> - **Weeks 4 and 5:** the k3s node, and `web` and `db` defined in `week-5/*.tf` with the image `week-4-web:<tag>`.
>
> Your repository is public (you checked what it exposes in Week 7), which makes criterion 3 a real audit question for you.
### Environment Checkpoint
 
Run these and save the output. They confirm the app, cluster, and Prometheus are healthy before you start scanning.
 
```bash
docker compose -f week-2/docker-compose.yml ps
curl -s http://localhost:<your Compose port>/health
k3d cluster list
kubectl get nodes
kubectl get pods
kubectl get pods -n monitoring
kubectl get networkpolicy
kubectl get serviceaccount flask-app
```
 
For Prometheus, run the port-forward in its own terminal (foreground, not backgrounded):
 
```bash
kubectl port-forward -n monitoring svc/prometheus-server <your Prometheus port>:80
```
 
Then in another terminal, run `week-5/pq.sh` with your `PROM_URL` and confirm it returns a result. If anything is not healthy, fix that first. A scan of a broken system produces meaningless evidence.

**Local VM:** your Compose stack has been stopped since Week 4, so use this block instead:

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -n monitoring
kubectl get networkpolicy
kubectl get serviceaccount default
curl -s http://$(kubectl get service web -o jsonpath='{.spec.clusterIP}'):8080/health
gh api repos/{owner}/{repo}/rulesets --jq '.[] | "\(.id) \(.name) \(.enforcement)"'
```

Use `kubectl port-forward --namespace monitoring service/prometheus-server 9090:80` and `./week-6/pq.sh 'up'` for Prometheus.

 
## Team Port Assignments
 
This track needs no new ports. Use the ports you already have:
 
> **Shared VM (RHEL, teams):** Compose port is 8080 + team number, the k3d load balancer is 8090 + team number (8091-8097 for teams 1-7), and your Prometheus port-forward is 9090 + team number. Use only your own ports.
 
> **Local VM (Ubuntu, individual):** There is no team number. The app is on port 8080 (the `web` Service address, or `kubectl port-forward --address 0.0.0.0 service/web 8080:8080` and http://localhost:8080 in your browser), and Prometheus on 9090.
 
## Tools Available
 
| Tool | What it's for | Verify (both OSs) | Ubuntu install |
|------|---------------|-------------------|----------------|
| [Trivy](https://trivy.dev/docs/latest/guide/) | Scan container images, Kubernetes manifests, and Dockerfiles for vulnerabilities and misconfigurations | `trivy --version` | Download the latest release (see below) |
| [gitleaks](https://github.com/gitleaks/gitleaks) | Find committed secrets in a repo and its full history | `gitleaks version` | Download the latest release (see below) |
| [kube-score](https://kube-score.com/) | Score Kubernetes manifests against best practices (probes, limits, security context) | `kube-score version` | Download the latest release (see below) |
| [checkov](https://www.checkov.io/) | Static policy checks for Dockerfiles, Kubernetes YAML, GitHub Actions, and OpenTofu files | `~/cybersecurity-governance-venv/bin/checkov --version` | `~/cybersecurity-governance-venv/bin/pip install checkov` (after creating the venv, see below) |
| [kubectl auth can-i](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_auth/kubectl_auth_can-i/) | Ask the cluster "can this identity do this action?" for RBAC verification | `kubectl auth can-i --help` | Already installed in earlier weeks |
| [jq](https://jqlang.org/manual/) | Filter JSON scan reports into counts and tables | `jq --version` | `sudo apt install -y jq` |
| [git](https://git-scm.com/doc) | Search history, check commit and branch rules | `git --version` | Already installed in earlier weeks |
| [Lynis](https://cisofy.com/documentation/lynis/) (Local VM only) | Audit the host OS against hardening checks | `lynis show version` | `sudo apt install -y lynis` |
 
> **Shared VM (RHEL, teams):** The system tools in this table (everything except Python packages in your own virtualenv) are pre-installed by the TA. Run the Verify command for each tool. If a command is not found, post in the course channel or tell the TA or professor before continuing. Do not install anything system-wide.
 
> **Local VM (Ubuntu, individual):** Your VM runs Ubuntu Server 26.04.1. You install tools yourself. Run the command in the last column, then the Verify command.
 
> **Shared VM (RHEL, teams):** Lynis is **not** provided on the shared VM and you must not install or run it there. It audits the whole host, and the host is shared by every team. The Local VM path covers it.
 
**Versions.** This table has no version numbers on purpose: use the current stable release of each tool, so these instructions keep working in future semesters. Record what you actually used. Run each Verify command and save the output to `capstone/cybersecurity-governance/TOOL-VERSIONS.txt`, then commit it. For Python packages, also save `~/cybersecurity-governance-venv/bin/pip freeze > capstone/cybersecurity-governance/requirements.txt` after you install them, so a teammate (or a future you) can rebuild the same environment.
 
**Single-binary installs (Ubuntu only).** For Trivy, gitleaks, and kube-score, look up the newest release tag, download the archive and its checksum file, check the hash, then install the binary. First define a small helper (paste it into your terminal):
 
```bash
# Print the newest release tag of a GitHub project, for example: latest_tag aquasecurity/trivy
# It follows the /releases/latest redirect, so it needs no login and no API key.
latest_tag() { curl -fsSLI -o /dev/null -w '%{url_effective}\n' "https://github.com/$1/releases/latest" | sed 's|.*/tag/||'; }
```
 
Then, for Trivy (gitleaks and kube-score follow the same pattern; their archive names are `gitleaks_<version>_linux_x64.tar.gz` from `gitleaks/gitleaks` and `kube-score_<version>_linux_amd64.tar.gz` from `zegl/kube-score`, where `<version>` has no leading `v`):
 
```bash
cd "$(mktemp -d)"
V=$(latest_tag aquasecurity/trivy)
curl -fLO "https://github.com/aquasecurity/trivy/releases/download/${V}/trivy_${V#v}_Linux-64bit.tar.gz"
curl -fLO "https://github.com/aquasecurity/trivy/releases/download/${V}/trivy_${V#v}_checksums.txt"
sha256sum --check --ignore-missing "trivy_${V#v}_checksums.txt"
tar -xzf "trivy_${V#v}_Linux-64bit.tar.gz" trivy
sudo install -m 0755 trivy /usr/local/bin/
```
 
If a download returns "404 Not Found", the project renamed its release file. Open the project's releases page, copy the exact file name, and adjust the command.
 
**Lynis (Local VM only).** After `sudo apt install -y lynis`, run `lynis audit system --quick` as your normal user. Do not use sudo to run the audit: without root, Lynis runs in non-privileged mode, which is enough for this course and avoids changing anything on your machine.
 
**Create your virtualenv.** You create this yourself. It lives in your own home directory, so no sudo and no system pip are needed.
 
```bash
python3 -m venv ~/cybersecurity-governance-venv
~/cybersecurity-governance-venv/bin/pip install --upgrade pip
~/cybersecurity-governance-venv/bin/pip install checkov
```
 
> **Shared VM (RHEL, teams):** Each person who runs the code creates their own virtualenv from the same package list (commit it as `requirements.txt`). Never use `sudo` or system `pip`.
 
> **Local VM (Ubuntu, individual):** Use the virtualenv above. Do not use `pip install` outside it.
 
These packages publish prebuilt wheels, so nothing should compile. If `pip` starts compiling something for more than a minute, stop and ask the TA. checkov is a large install, so check `df -h ~` first.
 
**Trivy note.** Trivy downloads its vulnerability database from the internet the first time it runs. That works from the host. It does not work from inside a container on the campus network, so run Trivy directly on the host, not in a container.
 
## Example Approaches
 
These are examples, not requirements. Combine them, change them, or ignore them.
 
**Framework-first.** Start with a short list of CIS-style controls (for example "containers do not run as root," "images have no known CRITICAL vulnerabilities," "no secrets in git," "default service account tokens are not mounted"). Write each as a yes/no test, run the test, and fill in the control matrix row by row. This demonstrates how an abstract standard becomes checkable, and it makes the matrix the spine of your whole deliverable.
 
**Scanner-first.** Run Trivy, gitleaks, kube-score, and checkov on everything you own, collect the raw findings, then map findings upward to controls and risks. Fix the cheapest high-severity items, rebuild, and rescan to show the before and after. This demonstrates triage: not every finding deserves the same response, and the risk register is where you explain why.
 
**Evidence-pipeline.** Build the one-command evidence collector first, with output going to a dated folder, and grow it as you add checks. Add a CI job that runs the cheap scans on every pull request. This demonstrates repeatability, which is what makes evidence credible to an external auditor.
 
## Track Tips and Common Trip-Ups

These notes come from where students usually get stuck. They do not change the success criteria; they help you reach them. Concepts are explained in [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) and commands in [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md). If a flag here does not work on your installed version, check the tool's `--help`.

**From control to evidence (criterion 1)**

Each row in `controls.csv` links a rule, a yes/no test, and a saved file. A worked example using one of the README's own controls (column names are a suggestion):

```csv
id,area,control,test,expected,status,evidence
C-01,Kubernetes workload,Flask containers do not run as root,Read runAsUser and runAsNonRoot from the Flask Deployment spec,runAsUser 1000 and runAsNonRoot true,Pass,evidence/2026-12-01-1030/c01-flask-securitycontext.txt
```

The evidence file holds the command and its output, for example `kubectl get deploy <flask deployment> -o jsonpath='{.spec.template.spec.securityContext}{"\n"}{.spec.template.spec.containers[*].securityContext}'`. Write the expected result before you run the test. A `Fail` you can explain is worth more than a `Pass` you cannot prove.

**Triage (criterion 2)**

- For each HIGH or CRITICAL finding, decide: **fix** (a fix exists and is cheap), **accept** (no fix, or the cost outweighs the risk; goes in the risk register with an owner), or **false positive** (write why).
- The cheapest image fix is often a newer base image tag. Rebuild with your team tag, import, rescan, and save both JSON reports.
- Count findings from a JSON report with jq, for example `jq '[.Results[]?.Vulnerabilities[]? | select(.Severity=="HIGH" or .Severity=="CRITICAL")] | length' report.json`. Trivy's `--ignore-unfixed` helps triage, but do not use it to hide findings in your before report.

**Secrets (criterion 3)**

- gitleaks changed its subcommands between versions. Run `gitleaks --help` on the installed version, scan the repo root so the full git history is included, write the report to a file, and use the redact option so no secret value lands in the report.

**Access and network (criterion 4)**

- Test a ServiceAccount, not yourself: `kubectl auth can-i get secrets --as=system:serviceaccount:default:flask-app`. Without `--as`, you are checking your own permissions.
- Write each prediction before you run the check, then save predicted and actual side by side.

**Local VM notes (criteria 2 and 4)**

- **Your identities for criterion 4.** There is no `flask-app` ServiceAccount. Test the account your `web` pods actually use, `system:serviceaccount:default:default`, plus one other, such as Prometheus's `system:serviceaccount:monitoring:prometheus-server` (check the name with `kubectl get serviceaccount -n monitoring`). Creating a dedicated ServiceAccount for `web` in `week-5/`, with its token not mounted, is a good remediation to record.
- **Your NetworkPolicy tests.** With no default-deny, the policy to test is `db-clients`. Predict, then test, at least two connections that should be blocked: a debug pod in `default` connecting to `db` on 5432, and a pod in another namespace doing the same. Then ask what a pod labeled `app: web` that is not your app would be allowed to do, and decide whether that belongs in the risk register. Adding a default-deny policy through `week-5/` is a valid remediation, but plan which allows `web`, `db-exporter`, and Prometheus scraping need first.
- **Scanning your manifests.** Your manifests are OpenTofu files, not YAML. `checkov -d week-5 --framework terraform` and `trivy config week-5` read them directly. kube-score needs YAML, so feed it the live objects: `kubectl get deploy web db -o yaml | kube-score score -`.
- **Your image** is `week-4-web:<the tag in week-5/web.tf>`. Remediation means a new tag, imported into k3s and deployed through a pull request; see [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) under "Shipping an app change".
- **You already have controls with evidence.** Your Week 8 gate and ruleset are controls; `gh pr checks` on a past red pull request and the ruleset JSON are their evidence.

**The evidence script (criterion 5)**

- Write every run to a new folder, for example `evidence/$(date +%Y-%m-%d-%H%M)/`.
- Scanners exit with a non-zero code when they find something. If your script uses `set -e`, one finding stops the whole run. Capture each scanner's exit code instead of letting it end the script.
- Time it with `time ./collect-evidence.sh` and save that output too.
- Risk score is likelihood times impact (1 to 25). Owners come from the Harborlight cast.

## Rules of the Road
 
- **Scan only what is yours.** Your own image, your own repo, your own cluster, your own ports. No nmap, port sweeps, or probing of the VM, the campus network, or another team's ports or cluster. Trivy, kube-score, checkov, and gitleaks read files and images you point them at; keep it that way.
- **Never use sudo on the shared VM.** Run k3d, kubectl, and helm without sudo (`sudo` cannot find `/usr/local/bin` tools anyway). Scanners also work without root.
- **No Lynis on the shared VM.** It audits the host and would measure (and could disturb) every team's work. Use it only on your own Local VM, as a normal user.
- **Real secrets stay out of the repo and the docs.** If gitleaks finds a real credential, tell your TA or professor and rotate it. In your reports, redact the value. For testing the scanner, use an obviously fake value in a scratch folder outside your repo, never in a commit.
- **Images are tagged `<name>:team<N>`** (for example `week-2-flask:team3`) and imported with `k3d image import <image>:team<N> -c <your cluster name>`. Local VM: use a new version tag such as `week-4-web:v3`, imported with `docker save` and `sudo k3s ctr images import` (there is no k3d).
- **Builds use host networking.** The campus firewall blocks container addresses from the internet, so build with `docker build --network host -t <image>:team<N> <dir>`. Do not use `build: network:` in Compose, and never disable firewalld or edit `/etc/docker/daemon.json`.
- **File permissions:** the VM uses `umask 077`, so edited files become mode 600. If a Dockerfile switches to a non-root user, use `COPY --chmod=644` (or `chmod 644` the files before building), or the container will not be able to read them.
- **Port-forwards run in the foreground** in their own terminal, never backgrounded with `&`. If one gets stuck: `pkill -u $USER -f "kubectl port-forward"`.
- **Any pod you add needs a NetworkPolicy allow** if it receives traffic; the default-deny will block it otherwise. Every pod you deploy needs memory `requests` and `limits`. A throwaway test pod that only sends traffic does not need an ingress allow, but still needs memory limits and must be deleted afterward.
- **Memory budget:** keep anything you add to the cluster under 128Mi total. Most of this track runs as CLI tools, not pods. Check with `kubectl top pods -A` and `free -h`. If `kubectl top` errors, tell the TA, and use `kubectl describe pod` (requests and limits) with `free -h` in the meantime.
- **Infrastructure code:** on the shared VM, do not run `tofu plan`, `tofu apply`, or `ansible-playbook`. If you change `infrastructure/flask.tf`, mark the step **[PREDICT ONLY]**, write your prediction in the Google Doc, and apply the live change with `kubectl`. Scanning the `.tf` file with checkov is fine; it only reads the file. Local VM students: your OpenTofu lives in `week-5/` and runs through your deploy pipeline; see the Local VM rules below.
- **Git on the shared VM:** `git switch -c <branch>`, `git push -u origin <branch>`, open a PR, wait for `build-and-push`, then merge. Never push to `main`.
- **Be honest in the evidence.** A `Fail` with a clear explanation is better than a `Pass` you cannot prove. Auditors check.

### Local VM (Ubuntu, individual) rules

The rules above that mention other teams, team numbers, k3d, `--network host`, firewalld, `/etc/docker/daemon.json`, `umask 077`, or [PREDICT ONLY] describe the Shared VM. On your own VM these apply instead:

- **Change the cluster through OpenTofu.** Anything that should last goes in `week-5/*.tf`, through a pull request, the Deploy gate, and `week-7/deploy.sh`. Throwaway test pods may use `kubectl apply`, and you delete them afterward. A change made with `kubectl` to an object OpenTofu manages is undone by the next deploy.
- **The gate checks your additions.** Every container you add in `week-5/` needs CPU and memory requests and limits and must not allow privilege escalation, or the `security scan` job fails and the ruleset blocks the merge.
- **New image, new tag.** Build a new tag (for example `:v1`, `:v2`), import it with `docker save` and `sudo k3s ctr images import` **before** the pull request that uses it, and never overwrite the tag that is running.
- **`master` and `protect-master` stay.** Work on branches and merge through pull requests. Do not delete the ruleset, remove its checks, or add a bypass.
- **No default-deny.** Only the `db` pods are protected (by `db-clients`). A pod you add accepts traffic from anywhere until a NetworkPolicy selects it.
- **sudo** is for package installs, `sudo install` of single binaries, and `sudo k3s ctr`. Run `kubectl`, `helm`, `tofu`, and `k6` as your normal user.
- **Memory budget still applies:** keep anything you add to the cluster under 128Mi total.
- **Lynis is allowed on your own VM,** as a normal user, as described in Tools Available.
- **Remediations ship through the pipeline.** A fix to `week-5/*.tf` or a new image tag goes through a pull request, the gate, and `deploy.sh`, so your evidence shows the gate passing on the fix.

## How to Run This Sprint
 
### Sprint Timeline (Weeks 11 to 14)

Both sections follow the same four weeks. The planning steps below still apply; this table says where your work should be at the end of each week. The full plan is in [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md).

| Week | By the end of the week |
|---|---|
| 11 Kickoff | Environment Checkpoint saved. Tools verified. First scans of your image, repo, and manifests saved. Control matrix drafted with at least 8 rows. QA test cases drafted. |
| 12 Build | Remediation with before and after scans. RBAC and NetworkPolicy tests with written predictions. Risk register started. Control matrix at 15 rows, each pointing to evidence. |
| 13 Consolidate and rebuild | Three policies, evidence script running in under 5 minutes, RUNBOOK and ARCHITECTURE, rebuild, and gap log. |
| 14 Present | Rebuild presentation (live on the Shared VM, recorded on the Local VM), running the evidence script and walking one control row to its evidence. |

### Shared VM (RHEL, teams)
 
**Step 1.** Sprint planning. Turn the five success criteria into GitHub issues on your project board. Split big ones (for example, one issue per scanner).
 
**Step 2.** Assign roles:
- **Scrum Master:** runs standups, keeps the board current, owns the retrospective.
- **System Admin:** owns the cluster checks (RBAC, NetworkPolicy), the image rebuild, and the evidence script.
- **Developers (2):** one owns scanners and remediation (Trivy, gitleaks, kube-score, checkov); one owns governance documents (control matrix, risk register, policies).
- **QA:** writes test cases in `docs/qa-report-final.md` **before** the implementation is done, then independently re-runs them.
**Step 3.** Mid-sprint check-in. Halfway through, the team shows Sam or the TA: the control matrix draft, at least one scan report, and the first risk register entries. Decide what gets cut if time is short.
 
**Step 4.** QA finishes `docs/qa-report-final.md` and the team writes `docs/final-sprint-retrospective.md` at the end of the sprint.
 
### Local VM (Ubuntu, individual)
 
You follow the same Sprint Timeline as the Shared VM teams, alone.

- **Week 11 milestone:** tools installed and verified, Environment Checkpoint saved, first scans of your image, repo, and OpenTofu files saved, control matrix drafted (at least 8 rows), your test cases written in `docs/qa-report-final.md`.
- **Week 12 milestone:** remediation shipped through the pipeline with before and after scans, RBAC and NetworkPolicy tests with written predictions, risk register started, control matrix at 15 rows with evidence. Optionally run Lynis on your own VM and add a short note on how host findings would appear in the matrix.
- **Week 13:** three policies, evidence script running in under 5 minutes, RUNBOOK and ARCHITECTURE, then rebuild on a new VirtualBox VM and keep a gap log (see the Rebuild section).
- **Week 14:** recorded presentation of your rebuild.
- You write your own test cases in the same QA file and sign as both author and tester. A short reflection (the "Individual Reflection" section of the retrospective file) replaces the team retrospective.

## Rebuild (Weeks 13 and 14)

Your work has to come back up from your repository alone. Start from a clean VM, clone your repo, and follow only your README, RUNBOOK, and scripts. Every step that fails or that your README forgot is a gap: write it in `capstone/cybersecurity-governance/REBUILD-GAPS.md`, fix it by pull request, and update the README. The full contract, the "what lives outside git" checklist, and the gap log template are in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md).

> **Shared VM (RHEL, teams):** In Week 13 your team takes a turn in a queue to test the rebuild on a separate VM, which is wiped between teams. In Week 14 you present live: the tools are pre-installed, and your team clones the repo and brings the app up from your README and setup instructions. There is no sudo on the rebuild or presentation VM, so every step your README asks someone to run must work without it. Your Ansible playbooks and any [PREDICT ONLY] code stay reference-only: do not run them, and make sure bringing the app up does not depend on them. Recreate your k3d cluster with the same `--subnet 100.97.<team>.0/24` flag you used before.

> **Local VM (Ubuntu, individual):** In Week 13 create a new VM in VirtualBox using the same steps you used for your original VM, clone your repo, and follow your README. Keep the original VM until the rebuild works. In Week 14 you present a recording of the rebuild. Start from the "Rebuild on a fresh VM" sections you wrote in your Week 7, 8, and 9 READMEs (k3s, OpenTofu variables, image import, deploy and backup cron lines), then add everything this track added. The full Local VM order is in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md). You have sudo on your own VM; every step that needs it must be in your README.

**Proof of life for this track:** the evidence script produces a new timestamped folder, and the statuses in `controls.csv` still match what the new evidence shows.

**Gaps this track usually finds:**
- The rebuilt Flask image does not exist on the new VM, so image scans fail until the build and import steps are documented.
- Trivy needs to download its vulnerability database on first run, which only works from the host.
- The evidence script assumes tools, paths, or a kube context that only existed on the old VM.
- Local VM: the remediated image tag was imported into k3s on the old VM only, so the first deploy on the new VM cannot start `web` until the build and import steps run.

## Deliverables
 
- Implementation committed under `capstone/cybersecurity-governance/` in your repo (merged via PR on the RHEL section)
- `capstone/cybersecurity-governance/RUNBOOK.md`: how to operate, check, and recover the implementation
- `capstone/cybersecurity-governance/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages): the tool versions you actually used (see Tools Available)
- `capstone/cybersecurity-governance/ARCHITECTURE.md` with an architecture diagram (Mermaid OK) showing VM, cluster, app, and new components
- A "How to run this" section added to the repo's top-level README so someone who clones the repo can bring the track implementation up
- `docs/qa-report-final.md` with your own test cases and results
- `capstone/cybersecurity-governance/REBUILD-GAPS.md`: the gap log from your Week 13 rebuild (template in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md))
- Shared VM: `docs/final-sprint-retrospective.md`; Local VM: a short reflection (the "Individual Reflection" section of the retrospective file)
- Final presentation/demo: 8-10 minutes for a team on RHEL (5 minutes for an individual on the Local VM). Show the demo from the command line: run the evidence script live, walk through one control row from the matrix to its evidence file, show a before/after scan, and explain one accepted risk in the risk register.
- Week 14 rebuild presentation: live on the Shared VM (clone the repo and bring the app up from your README), recorded on the Local VM. See the Rebuild section.
Track-specific items:
- `capstone/cybersecurity-governance/controls.csv` (control matrix)
- `capstone/cybersecurity-governance/risk-register.md`
- `capstone/cybersecurity-governance/policies/` with at least 3 one-page policies
- `capstone/cybersecurity-governance/evidence/` with before and after scan reports, the RBAC and NetworkPolicy test output, and the gitleaks report
- The evidence collection script (and, optionally, a CI job that runs the cheap scans)
## Discussion (add to Google Doc)
 
1. Elena asks, "Does a clean scan mean we are secure?" Using your own results, explain what your scans can and cannot tell an auditor.
2. Which finding did you choose to accept rather than fix? Who at Harborlight should sign off on that decision, and why them?
3. Your control matrix has a `Fail` row you could not fix before the deadline. How would you present it to the auditor so that it builds trust instead of destroying it?
4. Where does "shift left" (scanning in CI) end and "evidence for an auditor" begin? What does a CI check give you that a one-time scan does not?
5. Marcus says the platform cannot slow down releases. Which of your checks would you make a blocking gate, which would only warn, and why?
## Getting Help
 
Clarifications go to the TA or professor. There is no wiki for this track, so ask early rather than guessing what a requirement means.

**Before you ask:** work through [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) for about 30 minutes, then ask with the command, its full output, what you expected, and what you tried.

**Using AI:** AI assistants are welcome as a tool for learning how these tools work and how enterprises use them. Do not rely on them to produce your work: understand every line you commit, verify AI answers against the tool's documentation, never paste AI output into evidence files, and never paste secrets into an AI tool. See [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md).
