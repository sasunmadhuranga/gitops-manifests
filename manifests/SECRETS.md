# ── HOW TO CREATE SECRETS ─────────────────────────────────────────────────────
#
# NEVER commit real passwords to Git. Create these secrets manually with kubectl.
# They are listed here for reference only.
#
# ─────────────────────────────────────────────────────────────────────────────
# 1. Grafana admin credentials
# ─────────────────────────────────────────────────────────────────────────────
#
#   kubectl create secret generic grafana-admin-secret \
#     --namespace monitoring \
#     --from-literal=admin-user=admin \
#     --from-literal=admin-password='YOUR_STRONG_PASSWORD'
#
# ─────────────────────────────────────────────────────────────────────────────
# 2. Gmail App Password for Alertmanager
# ─────────────────────────────────────────────────────────────────────────────
# Gmail requires an App Password (not your account password).
# Create one at: https://myaccount.google.com/apppasswords
# (Google Account → Security → 2-Step Verification → App passwords)
#
#   kubectl create secret generic gmail-password \
#     --namespace monitoring \
#     --from-literal=password='YOUR_GMAIL_APP_PASSWORD'
#
# ─────────────────────────────────────────────────────────────────────────────
# 3. ArgoCD Notifications email password
# ─────────────────────────────────────────────────────────────────────────────
# Same Gmail App Password, but stored in the argocd namespace.
#
#   kubectl create secret generic argocd-notifications-secret \
#     --namespace argocd \
#     --from-literal=email-password='YOUR_GMAIL_APP_PASSWORD'
#
# ─────────────────────────────────────────────────────────────────────────────
# Run all three BEFORE applying the monitoring manifests.
# The monitoring stack will fail to start if these secrets are missing.
