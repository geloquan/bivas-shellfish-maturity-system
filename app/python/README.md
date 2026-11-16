# `python/`

Python code for running a YOLO (`.pt`) model from the Laravel project. It sits outside `app/` because it isn't PHP.

## Layout

```
python/
├── detect.py            # script Laravel calls to run the model on an image
├── best.pt              # YOLO model weights
├── requirements.txt     # Python dependencies
└── .venv/               # virtualenv (not committed)
```

## `detect.py`

Takes an image path as its argument, runs the model, and prints the result as JSON.

```bash
python detect.py /path/to/image.jpg
```

Rules for the script:

- Print only JSON to stdout, since Laravel runs `json_decode` on it. Send logs and warnings to stderr.
- Load the model with an absolute path so it works regardless of where PHP runs it from:
  ```python
  os.path.join(os.path.dirname(__file__), "best.pt")
  ```

## `best.pt`

The model file. Large model files are better handled with Git LFS, or added to `.gitignore` and deployed separately.

## `requirements.txt`

Python dependencies for the script.

## Virtualenv

Use a virtualenv in `python/.venv` and call its interpreter instead of the system one. Add `.venv` to `.gitignore`.

## Calling it from Laravel

```php
use Illuminate\Support\Facades\Process;

$result = Process::run([
    base_path('python/.venv/bin/python'),
    base_path('python/detect.py'),
    storage_path('app/uploads/photo.jpg'),
]);

$detections = json_decode($result->output(), true);
```

## Limitations

The script reloads the model on every call, so each call takes a few seconds. This is fine for prototypes and low volume. If speed or traffic becomes a problem, the model can move into a long-running service such as FastAPI; only the call on the Laravel side changes.
