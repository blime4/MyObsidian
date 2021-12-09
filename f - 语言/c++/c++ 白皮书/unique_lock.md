

`unique_lock` 是一个 [[RAII]] 对象，确保用户不会忘记在这个 `mutex` 上调用 `unlock()`。