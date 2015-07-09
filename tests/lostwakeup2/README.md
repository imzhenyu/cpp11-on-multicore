This project tests `LightweightSemaphore` for lost wakeups.

The original implementation of `LightweightSemaphore::wait(int timeout_milliseconds)` looked like this:

    bool wait(int timeout_milliseconds)
    {
        if (!tryWait())
            return m_sema.wait(timeout_milliseconds);
        else
            return true;
    }

Fixed as this:

    bool wait(int timeout_milliseconds)
    {
        if (tryWait())
            return true;
        int oldCount = m_count.fetch_sub(1, std::memory_order_acquire);
        if (oldCount > 0)
            return true;
        if (m_sema.wait(timeout_milliseconds))
            return true;
        signal(); // restore the substracted count
        return false;
    }

The test in this folder can both reproduce the original bug and validate the fix.
Build and run it using the same steps as `tests/lostwakeup`.
