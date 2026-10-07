# Session 15 - Helm Homework

Helm v4.3.0, namespace `s15`.

# Task 1: Helm Commands

**helm repo** - add, update and list chart repositories.

![](screenshots/01-repo.png)

**helm search** - `search repo` looks in added repos, `search hub` looks in Artifact Hub.

![](screenshots/02-search.png)

**helm create** - generates a starter chart (Chart.yaml, values.yaml, templates). Chart is in [helm-commands/mychart](helm-commands/mychart).

![](screenshots/03-create.png)

**helm install** - renders templates with values and creates a release (revision 1).

![](screenshots/04-install.png)

**helm list / helm status** - list releases and show status of one release.

![](screenshots/05-list-status.png)

**helm get** - `get values` shows the values I passed (`--all` shows everything), `get manifest` shows the final YAML applied.

![](screenshots/06-get.png)

**helm upgrade / helm history** - upgrade changes the release and adds a new revision. History shows all revisions.

![](screenshots/07-upgrade.png)

**helm rollback** - goes back to an old revision. It creates a new revision ("Rollback to 1"), it does not delete history.

![](screenshots/08-rollback.png)

**helm uninstall** - deletes all resources of the release.

![](screenshots/09-uninstall.png)

# Task 2: Helm Rollback

Chart: [07-install-upgrade/app-chart](07-install-upgrade/app-chart) (nginx, default tag 1.24)

**Install -> verify** (rev 1, nginx 1.24)

![](screenshots/10-rb-install.png)

**Upgrade -> verify** (rev 2, nginx 1.25)

![](screenshots/11-rb-upgrade1.png)

**Upgrade again -> verify** (rev 3, nginx 1.27, 3 replicas)

![](screenshots/12-rb-upgrade2.png)

**Rollback -> verify** (back to rev 2: nginx 1.25, 1 replica)

![](screenshots/14-rb-rollback.png)

The rollback created revision 4 with the same values as revision 2 (`image.tag=1.25`). Image and replica count both went back.

# Task 3: Mini Project - Notes App chart

Chart: [mini-project/notes-chart](mini-project/notes-chart) (Chart.yaml, values.yaml, values-prod.yaml, templates for deployment, service, configmap)

**Lint and render**

![](mini-project/screenshots/01-lint-template.png)

**Install (development values)** - 1 replica, nginx 1.24, ENVIRONMENT=development, NodePort 30090

![](mini-project/screenshots/02-install.png)

**Upgrade with production values** - 3 replicas, nginx 1.25, ENVIRONMENT=production

![](mini-project/screenshots/03-upgrade-prod.png)

**Bad upgrade** - broken image tag. The new pod is in ImagePullBackOff, but the old pods keep running because of the rolling update, so the app is still up.

![](mini-project/screenshots/04-bad-upgrade.png)

**Rollback to revision 2** - the broken pod is terminating and the 3 good pods stay.

![](mini-project/screenshots/05-rollback.png)

**Clean up**

![](mini-project/screenshots/06-uninstall.png)

The release is removed. The pod from the bad upgrade is still shutting down (Terminating) and is deleted a few seconds later.
