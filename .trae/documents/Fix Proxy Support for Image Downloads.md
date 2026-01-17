I will fix the image download issues by ensuring the proxy is correctly applied in all download paths.

1.  **Update `src/api/routes.py`**:
    *   Refactor `retrieve_image_data` to use `generation_handler.file_cache.download_and_cache`. This will unify the download logic and ensure it uses the `FileCache`'s proxy handling (which I will also improve).
    *   This fixes the issue where `routes.py` was ignoring proxies entirely.

2.  **Update `src/services/file_cache.py`**:
    *   Enhance `download_and_cache` to check for system environment variables (`HTTP_PROXY`, `HTTPS_PROXY`) if `proxy_manager` does not return a configured proxy.
    *   This ensures that even if the database config is missing/disabled, the system proxy (or the one user specified via env) is used.

3.  **Update `src/core/config.py`**:
    *   Add a method to retrieve the proxy configuration directly from `setting.toml` as a fallback mechanism, just in case the database isn't initialized or sync'd yet. (Actually, reading env vars is cleaner and standard, I'll stick to env vars and maybe `setting.toml` check).

I will start by modifying `src/api/routes.py`.