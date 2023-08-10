## Setup Deploy Django Project

## Table of content
1. Setup Django Project
    - Add env.py to connect gcp secret
    - Add Dockerfile & docker-compose.yml
    - Add yaml file
        - createsuperuser
        - cloudbuild
        - collectstatic 
    - Add createuser.py in "user / management / command"
2. Setup Sentry 
3. Setup GCP (Google Cloud Platform)

## Setup Django Project
1 Add env.py add in the same directory of settings.py to connect gcp secret
```python
import io
import os
import google


from dotenv import load_dotenv
from google.cloud import secretmanager


def load_env(env_file):
    try:
        _, os.environ["GOOGLE_CLOUD_PROJECT"] = google.auth.default()
    except google.auth.exceptions.DefaultCredentialsError:
        pass

    if os.path.isfile(env_file):
        # Use a local secret file, if provided
        print(f"load env from {env_file}")
        load_dotenv()

    elif os.environ.get("GOOGLE_CLOUD_PROJECT", None):
        # Pull secrets from Google Secret Manager
        print(f"load env from google secret manager")

        project_id = os.environ.get("GOOGLE_CLOUD_PROJECT")
        client = secretmanager.SecretManagerServiceClient()
        settings_name = os.environ.get("SETTINGS_NAME", "dobybot_settings")
        name = f"projects/{project_id}/secrets/{settings_name}/versions/latest"
        payload = client.access_secret_version(name=name).payload.data.decode("UTF-8")
        load_dotenv(stream=io.StringIO(payload))

    else:
        print("No local .env or GOOGLE_CLOUD_PROJECT detected. No secrets found.")
```

2. Add Dockerfile & docker-compose.yml
``` yaml                                                                                 
steps:
  - id: "create super user"
    name: "gcr.io/google-appengine/exec-wrapper"
    args:
      [
        "-i", "gcr.io/$PROJECT_ID/${_SERVICE_NAME}",
        "-s", "${PROJECT_ID}:${_REGION}:${_CLOUD_SQL_INSTANCE_NAME}",
        "-e", "SETTINGS_NAME=${_SECRET_SETTINGS_NAME}",
        "--", "python", "manage.py", "collectstatic", "--no-input",
      ]

substitutions:
  _CLOUD_SQL_INSTANCE_NAME: newcpd-1
  _REGION: asia-southeast1
  _SERVICE_NAME: uat-ez-online-trainging-backend
  _SECRET_SETTINGS_NAME: uat-ez-online-trainging-backend

```
