程序清单4-3 通过一个私有锁，来保护状态  
```
public class PrivateLock {
	private final Object myLock = new Object();
	@guardedBy("myLock") Wiget wiget;

	void someMethod() {
		synchronized(myLock) {
			//访问或者修改Widget的状态
		}
	}
}
```
上面的代码中，使用private锁来保护状态。  
私有的锁，把锁封装起来，客户代码如果想要获取锁，需要通过公开的方法访问锁，这样机会进入到同步的策略当中。  


4.2.2示例：车辆追踪 
使用Java监视器模式来创建车辆追踪器。每个车由String对象来标识，
```
Map<String, Point> locations = vehicles.getLocations();
for(String key : locations.keySet()) {
	renderVehicle(key, locations.get(key));
}

//获取数据，修改车辆位置
void vehicleMoved(VehicleMovedEvent evt) {
	Point loc = evt.getNewLocation();
	vehicle.setLocation(evt.getVehicleId(), loc.x, loc.y);
}

@ThreadSafe 
public class MonitorVehicleTracker {
	@GuardedBy("this")
	private final Map<String, MutablePoint> locations;

	public MonitorVehicleTracker(Map<String, MutablePoint> locations) {
		this.locations = deepCopy(locations);
	}

	public synchronized Map<String, MutablePoint> getLocations() {
		return deepCopy(locations);
	}

	public synchronized MutablePoint getLocation(String id) {
		MutablePoint loc = locations.get(id);
		return loc == null ? null : new MutablePoint(loc);
	}

	public synchronized void setLocation(String id, int x, int y) {
		Mutable loc = locations.get(id);
		if(loc == null) {
			throw new IllegalArgumentException("No such ID" + id);
		}
		loc.x = x;
		loc.y = y;
	}

	private static Map<String, Mutable> deepCopy(Map<String, MutablePoint> m) {
		Map<String, Mutable> result = new HashMap<String, Mutable>();
		for(String id : m.keySet()) {
			result.put(id, new MutablePoint(m.get(id)));
		}
		return Collections.unmodifiableMap(result);

	}
}

//不是线程安全的可变化Point类
public class MutablePoint {
	public int x, y;

	public MutablePoint() {
		x = 0;
		y = 0;
	}
	public MutablePoint(MutablePoint p) {
		x = p.x;
		y = p.y;	
	}
}


```

4.3线程安全的委托

不可变的Point类，保证线程安全。因为不可变，所以返回location的时候不需要复制。
```
@Immutable
public class Point {
	public int x, y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```

在DelegatingVehicleTracker中没有使用显示的同步，而是用ConcurrentHashMap来管理状态，Map的键和值都不可以改变。  
```
public class DelegatingVehicleTracker {
	private final ConcurrentHashMap<String, Point> locations;
	private final Map<String, Point> unmodifiableMap;

	public DelegatingVehicleTracker(Map<String, Point> points) {
		locations = new ConcurrentHashMap<String, Point>(points);
		unmodifiableMap = Collections.unmodifiableMap(locations);
	}

	public Map<String, Point> getLocations() {
		return unmodifiableMap;
	}

	public Point getLocation(String id) {
		return location.get(id);
	}

	public void setLocation(String id, int x, int y) {
		if(locations.replace(id, new Point(x, y)) == null) {
			throw new IllegalArgumentException("invalid vehicle name: " + id);
		}
	}

	//Map的内容是不可变的，只需要复制Map结构，不用复制它的内容。  
	//只返回一个HashMap，因为getLocation不能保证返回一个线程安全的Map
	public Map<String, Point> getLocation() {
		return Collections.unmodifiableMap(new HashMap<String, Point>(locations));
	}
}
```

4.3.2独立的状态变量  
把线程安全性委托各多个独立的状态变量，只要这些变量之间中相互独立的，他们组合成的类，可以保证安全性。  

下面的代码中，允许用户程序，用监听器监听鼠标和键盘事件。鼠标事件监听器和键盘事件监听器之间不存在关系，相互独立。
VisualComponent使用CopyOnWriteArrayList来保存每个监听器列表，它是线程安全的。每个状态之间不存在耦合关系，
所以VisualComponent把线程安全性委托给mouseListener和keyListener对象。 

```
public class VisualComponent {
	private final List<KeyListener> keyListeners = new CopyOnWriteArrayList<KeyListener>();
	private final List<MouseListener> keyListeners = new CopyOnWriteArrayList<MouseListener>();

	public void addKeyListener(KeyListener listener) {
		keyListeners.add(listener);
	}
	public void addMouseListener(MouseListener listener) {
		mouseListeners.add(listener);
	}
	public void removeKeyListener(KeyListener listener) {
		keyListeners.remove(listener);
	}
	public void removeMouseListener(MouseListener <li></li>istener) {
		mouseListeners.remove(listener);
	}
}
```

4.3.5发布状态的车辆追踪器
使用可变化而且线程安全的Point类。  
```
@ThreadSafe
public class SafePoint {
	@GuardedBy("this") private int x, y;

	private SafePoint(int [] a) {
		this(a[0], a[1]);
	}

	public SafePoint(SafePoint p) {
		this(p.get());
	}

	public SafePoint(int x, int y) {
		this.x = x;
		this.y = y;
	}

	public synchronized int[] get() {
		return new int[] {x, y};
	}

	public synchronized void set(int x, int y) {
		this.x = x;
		this.y = y;
	}
}

@ThreadSafe
public class PublishingVehicleTracker {
	private final Map<String, SafePoint> locations;
	private final Map<String, SafePoint> unmodifiableMap;

	public PublishingVehicleTracker(Map<String, SafePoint> locations) {
		this.locations = new ConcurrentHashMap<String, SafePoint>(locations);
		this.unmodifiableMap = Collections.unmodifiableMap(this.locations);
	}

	public Map<String, SafePoint> getLocation() {
		return unmodifiableMap;
	}

	public safePoint getLocation(String id) {
		return locations.get(id);
	}

	public void setLocation(String id, int x , int y) {
		if(!locations.containsKey(id)) {
			throw new IllegalArgumentException("Invalid vehicle name: " + id);
		}
		locations.get(id).set(x, y);
	}

}
```

//扩展Vector，并增加一个“如果没有就添加”的方法
```
@ThreadSafe
public class BetterVector<E> extends Vector<E> {
	public synchronized boolean putIfAbsent(E x) {
		boolean absent = !contains(x);
		if(absent) {
			add(x);
		}
		return absent;
	}
}
```

//线程不安全的做法
```
@NotThreadSafe
public class ListHelper<E> {
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
	...
	public synchronized boolean putIfAbsent(E x) {
		boolean absent = list.!contains(x);
		if(absent) {
			add(x);
		}
		return absent;
	}
}
```
//线程安全的做法，通过客户端加锁实现
```
@ThreadSafe
public class ListHelper<E> {
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
	...
	public  boolean putIfAbsent(E x) {
		synchronized (list) {
			boolean absent = list.!contains(x);
			if(absent) {
				add(x);
			}
			return absent;
		}
		
	}
}
```

//组合的方式，添加一个原子操作 
```
@ThreadSafe
public class ImprovedList<T> implements List<T> {
	private final List<T> list;

	public ImprovedList(<List<T> list>) {
		this.list = list;
	}
	public synchronized boolean putIfAbsent(T x) {
		boolean contains = list.contains(x);
		if(contains) {
			list.add(x);
		}
		return !contains;
	}
	...
}

```