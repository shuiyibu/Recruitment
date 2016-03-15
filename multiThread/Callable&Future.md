
```
public interface Callable<V>
{
	V call() throws Exception;
}
```

```
public interface Future<V>
{
	V get() throws ...;
	V get(long timeout, TimeUnit unit) throws...;
	boolean cancel(boolean mayInterrupt);
	boolean isCancelled();
	boolean isDone();

}
```
