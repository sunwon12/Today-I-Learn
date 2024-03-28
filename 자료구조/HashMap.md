#### **HashMap은 Key와 Value의 묶음모음으로 이루어져 있는 자료구조입니다.**

### **HashMap 생성법**               

-    **HashMap<String, String\> h1 = new HashMap<String, String\>( );**         _// 기본 capacity:16, load factor:0.75_
-    **HashMap<String, String\> h2 = new HashMap<String, String\>(20);**       _// capacity:20으로 설정_
-    **HashMap<String, String\> h3 = new HashMap<String, String\>(20, 0.8);** _// capacity:20, load factor:0.8로 설정_
-    **HashMap<String, String\> h4 = new HashMap<String, String\>(h1);**      _// 다른 Map(h1)의 데이터로 초기화_

c capacity는 데이터 저장 용량, load factor는 데이터 저장공간을 추가로 확보해야 하는 시점을 지정합니다. load factor 0.8 은 저장공간이 80% 채워져 있을 경우 추가로 저장공간을 확보합니다. 

### **HashMap 함수**                         

**1) 데이터 추가**

1.  **V put(K key, V value) :** key와 value를 저장합니다. 
2.  **void putAll(Map<? extends K, ? extends V> m) :** Map m의 데이터를 전부 저장합니다.
3.  **V putIfAbsent(K key, V value) :** 기존 데이터에 key가 없으면  key와 value를 저장합니다. 

**2) 데이터 삭제**

1.  **void clear( ) :** 모든 데이터를 삭제합니다. 
2.  **V remove(Object key) :** key와 일치하는 기존 데이터( key와 value)를 삭제합니다. 
3.  **boolean remove(Object key, Object value) :** key와 value가 동시에 일치하는 데이터를 삭제합니다. 

**3) 데이터 수정**

1.  **V replace(**K key, V value**) :** key와 일치하는 기존 데이터의 value를 변경합니다. 
2.  **V replace(**K key, V oldValue****, V newValue******) :** key와 oldValue가 동시에 일치하는 데이터의 value를 newValue로 변경합니다. 

**4) 데이터 확인**

1.  **boolean containsKey(Object key) :** key와 일치하는 데이터가 있는지 여부를 반환합니다. (있으면 true)
2.  **boolean containsValue(Object value) :** value가 일치하는 데이터가 있는지 여부를 반환합니다. (있으면 true)
3.  **boolean isEmpty( ) :** 데이터가 빈 상태인지 여부를 반환합니다. (빈 상태면 true)
4.  **int size( ) :** key-value 맵핑 데이터의 개수를 반환합니다. 

**5) 데이터 반환**

1.  **V get(Object **key**) :** key와 맵핑된 value값을 반환합니다. 
2.  **V getOrDefault(Object **key, V defaultValue**) :** key와 맵핑된 value값을 반환하고 없으면 defaultValue값을 반환합니다.
3.  **Set<Map.Entry<K, V>> entrySet( ) :** 모든 key-value 맵핑 데이터를 가진 Set 데이터를 반환합니다. 
4.  **Set<K> keySet( ) :** 모든 key 값을 가진 Set 데이터를 반환합니다. 
5.  **Collection<V> values( ) :** 모든 value 값을 가진 Collection 데이터를 반환합니다.
