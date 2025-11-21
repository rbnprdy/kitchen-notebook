# Refactor Locking

Locking functionality used in flow is pretty brittle and I think relies on incorrect assumptions about file system guarantees. Should probably use a lightweight database for locking instead?
