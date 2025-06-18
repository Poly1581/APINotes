# Sentry

## Integration

### Installation

Install for django with `pip install "sentry-sdk[django]`.
Install for celery with `pip install sentry-sdk[celery]`.

Note: Need to add to docker compose to ensure sentry is included in docker image.

### Versions and Support
Sentry version 2.0+ support Django 1.11+ and Python 3.6+.
If support for older Django/Python versions is required, use version 1.x.

### Usage

#### Python / Django

```python
import sentry_sdk

sentry_sdk.init(
    dsn = "https://KEY",        # Sentry url with project and key
    traces_sample_rate = 1.0,   # Error sample rate (percentage)
    release = "myapp@1.0.0",    # Tag errors with release version (Semantic version)
)
```

Note: install before anything else to ensure all errors are logged/captured.
Note: configuration should be done in settings.py file.
Note: Release version cannot:
    - Contain
        - Newlines
        - Tabulator characters
        - Forward or back slashes
    - Be entirely
        - Periods
        - Double periods
        - Spaces
    - Exceed 200 characters

[Semantic Versioning](https://semver.org/) or the Git commit SHA is recommended.
Prefixing with project specific identifier is suggested (e.g. "PACKAGE@VERSION").
Release can be controlled with local environment variable set up during build process (DOCKER).

#### Celery

If celery package is in dependencies, Celery integration will be enabled automatically.
If celery is configured to use the same settings as [config_from_object](https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html), there is no need to initialize Celery SDK.

To set options on CleryIntegration, add to `sentry_sdk.init()` as follows:
```python
import sentry_sdk
from sentry_sdk.integrations.celery import CeleryIntegration

sentry_sdk.init(
    integrations=[
        CeleryIntegration(
            # OPTIONS GO HERE
            monitor_beat_tasks=True
        )
    ]
)
```

Note: Options are available [here](https://docs.sentry.io/platforms/python/integrations/celery/#options).
Note: Celery 4.0+ with Python 3.6+ are supported.

#### Chrons

Chrons allow monitoring of uptime and performance of celery tasks.
Initialize sentry after celery beat schedule.

If beat is running in worker process (i.e. `-B/--beat` option), initialize in either `celeryd_init` or `beat_init`.
If beat is running in seperate process, initialize in BOTH `celeryd_init` and `beat_init`.

Make sure to set `monitor_beat_tasks=True` in `CeleryIntegration` (`sentry_sdk.init`).
Beat and worker services are available at [sentry.io/chrons](https://sentry.io/chrons).

Task monitoring can also be achieved with the `@sentry_sdk.monitor` decorator:
```python
@app.task
@sentry_sdk.monitor(monitor_slug="<monitor-slug>")
def celery_task():
    # TASK HERE
```
Note: need to supply a `monitor_slug` of a monitor created on sentry.io

### Before Send

`before_send` is a callback that allows payload modification before sending events to sentry.
Investigate using `before_send` to strip sensitive information.

```python
from sentry_sdk.types import Event, Hint

def before_send(event: Event, hint: Hint) -> Event | None:
    # Modify event

sentry_sdk.init(
    before_send = before_send
)
```

Typically, a `Hint` holds the original exception - see [hints](https://docs.sentry.io/platforms/python/configuration/filtering/#using-hints)
### API

```python
# Add breadcrumb
sentry_sdk.add_breadcrumb(category="CATEGORY", message="MESSAGE", level="LEVEL")

# Set tag (key-value pair)
sentry_sdk.set_tag("KEY", VALUE)
```

## CLI
A command line interface is available at [CLI](https://docs.sentry.io/product/cli)

## Notes
- Sentry grouping is modifiable.
- Error, stack trace, code, and local variables are all stored.
- Consider using data scrubbing to filter sensitive information.
- "Breadcrumbs" are events that happened leading up to error (useful for debug).

