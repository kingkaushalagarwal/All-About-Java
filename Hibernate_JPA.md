# Mock Rest Template 
[Ref](https://www.baeldung.com/spring-mock-rest-template)
```java
@Service
public class EmployeeService {
    
    @Autowired
    private RestTemplate restTemplate;

    public Employee getEmployee(String id) {
	ResponseEntity resp = 
          restTemplate.getForEntity("http://localhost:8080/employee/" + id, Employee.class);
        
	return resp.getStatusCode() == HttpStatus.OK ? resp.getBody() : null;
    }
}

@RunWith(MockitoJUnitRunner.class)
public class EmployeeServiceTest {

    @Mock
    private RestTemplate restTemplate;

    @InjectMocks
    private EmployeeService empService = new EmployeeService();

    @Test
    public void givenMockingIsDoneByMockito_whenGetIsCalled_shouldReturnMockedObject() {
        Employee emp = new Employee(“E001”, "Eric Simmons");
        Mockito
          .when(restTemplate.getForEntity(
            “http://localhost:8080/employee/E001”, Employee.class))
          .thenReturn(new ResponseEntity(emp, HttpStatus.OK));

        Employee employee = empService.getEmployee(id);
        Assert.assertEquals(emp, employee);
    }
}
```

# Hibernate Caching:-

## First level cache:- 

	The first level of caching is the persistence context. The JPA Entity Manager maintains a set of Managed Entities in the Persistence Context.
	The Entity Manager guarantees that within a single Persistence Context, for any particular database row, there will be only one object instance.
	acts within the scope of Transaction.
	enable bydeafult
	
	JPA -  This is by default cache in JPA. And IT is per session. every session has a cache in the memory and ot tries to retrive the data from there utill eviction.
	we can evict the the cache entry by -
	Session session = entityManger.unwrap(Session.class);
	..
	Product product = repo.findById(1l);
	session.evict(product);
	
	
## Second Level Cache:-

	If L2 caching is enabled, entities not found in persistence context, will be loaded from L2 cache, if found.
	we need to use EhCache  by adding dependency ->	hibernate-ehcache
	
	1. enable second level cache
		spring.jpa.properties.hibernate.cache.use_second_level_cache= true
	
	2. specify the chaching framework- ehcache
		spring.jpa.properties.hibernate.cache.region.factory_class = EhCacheRegionFactory
		
	3. Only cache what I tell you to cache. - i.e data which will not change between transactions
		spring.jpa.properties.javax.persistence.sharedCache.mode = Enabale_Selective
	
	4. What data to cache.
	Add @Cacheable to the entity you want to enable. like Entity Course.
	Or
	Make Entry cacheable
	@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
	//Good practise to implements serializable;
	
	The advantages of L2 caching are:

	avoids database access for already loaded entities
	faster for reading frequently accessed  unmodified entities
	
	The disadvantages of L2 caching are:
	memory consumption for large amount of objects
	Stale data for updated objects
	Concurrency for write (optimistic lock exception, or pessimistic lock)
	Bad scalability for frequent or concurrently updated entities
	
***************************************************************************8
## Eager Vs LAzy:
	
	Use Lazy fetching mostly.
	Remeber that all mapping *toOne(@ManytoOne or @OneToOne) are Eager by deafult.
	Questions:- Do I need to have details of Entity 2 everytime I fetch Entity 1. If Yes- Eager, Otherwise - Lazy
	
*****************************************************
## N+1 Problems:
	Hibernate executes too many(exactly N+1) small select query to load data.
	Solution:-
	1) EntityGraph
	2) JOIN FETCH

# *JPA*

## Hibernate-identifiers:

	Identifiers in Hibernate represent the primary key of an entity. This implies the values are unique so that they can identify a specific entity, that they aren't null and that they won't be modified.
	1) Simple Identifier - The most straightforward way to define an identifier is by using the @Id annotation.
	2) Generated Identifier
	3) Composite Identifier
	4) Derived Identifier

## What are Generators in JPA?

	Generators are used to generater primary key in a table.
		or 
	If we want the primary key value to be generated automatically for us, we can add the @GeneratedValue annotation.
	If we don't specify a value explicitly, the generation type defaults to AUTO.
	4 types of Generators:
	1) GenerationType.AUTO
	2) GenerationType.IDENTITY
	3) GenerationType.SEQUENCE
	4) GenerationType.TABLE
	5) Custom Generator
	
	GenerationType is enum in JPA.
	How to do this?
	eg:
	@ID
	@GeneratedValue(strategy = GenerationType.AUTO)
	int id;
	
	1.  Auto Generation:-
		If you are using  the default generation type , the persistance provider will determine values based on the type of primary key attribute.
		This can be Numeric or UUID.
		For numeric values, the generation is based on a sequence or table generator, while UUID values will use the UUIDGenerator.
		An interesting feature introduced in Hibernate 5 is the UUIDGenerator. To use this, all we need to do is declare an id of type UUID with @GeneratedValue annotation:
		eg:-
			@Id
			@GeneratedValue
			private UUID courseId;
 		// Hibernate will generate an id of the form “8dd5f315-9788-4d00-87bb-10eed9eff566”.
		
	2.  Identity Generation:-
	This type of generation relies on the IdentityGenerator which expects values generated by an identity column in the database, meaning they are auto-incremented.
	 
	 @GeneratedValue (strategy = GenerationType.IDENTITY)
   		 private long studentId;
	
	One thing to note is that IDENTITY generation disables batch updates.
	
	3.  SEQUENCE Generation:-
	To use a sequence-based id, Hibernate provides the SequenceStyleGenerator class.
	To customize the sequence name, we can use the @GenericGenerator annotation with SequenceStyleGenerator strategy:
	
		@Id
		@GeneratedValue(generator = "sequence-generator")
		@GenericGenerator(
		  name = "sequence-generator",
		  strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",
		  parameters = {
			@Parameter(name = "sequence_name", value = "user_sequence"),
			@Parameter(name = "initial_value", value = "4"),
			@Parameter(name = "increment_size", value = "1")
			}
		)
    	private long userId;
	
	In this example, we've also set an initial value for the sequence, which means the primary key generation will start at 4.
	
	4. TABLE Generation: 
	The TableGenerator uses an underlying database table that holds segments of identifier generation values.
		@Id
		@GeneratedValue(strategy = GenerationType.TABLE, 
		  generator = "table-generator")
		@TableGenerator(name = "table-generator", 
		  table = "dep_ids", 
		  pkColumnName = "seq_id", 
		  valueColumnName = "seq_value")
		private long depId;
	
	The disadvantage of this method is that it doesn't scale well and can negatively affect performance.
	
	5. Custom Generator:-
	If we don't want to use any of the out-of-the-box strategies, we can define our custom generator by implementing the IdentifierGenerator interface.
```java
	public class MyGenerator 
  	implements IdentifierGenerator, Configurable {
 
    private String prefix;
 
    @Override
    public Serializable generate(
      SharedSessionContractImplementor session, Object obj) 
      throws HibernateException {
 
        String query = String.format("select %s from %s", 
            session.getEntityPersister(obj.getClass().getName(), obj)
              .getIdentifierPropertyName(),
            obj.getClass().getSimpleName());
 
        Stream ids = session.createQuery(query).stream();
 
        Long max = ids.map(o -> o.replace(prefix + "-", ""))
          .mapToLong(Long::parseLong)
          .max()
          .orElse(0L);
 
        return prefix + "-" + (max + 1);
    }
 
    @Override
    public void configure(Type type, Properties properties, 
      ServiceRegistry serviceRegistry) throws MappingException {
        prefix = properties.getProperty("prefix");
    }
}

@Entity
public class Product {
 
    @Id
    @GeneratedValue(generator = "prod-generator")
    @GenericGenerator(name = "prod-generator", 
      parameters = @Parameter(name = "prefix", value = "prod"), 
      strategy = "com.baeldung.hibernate.pojo.generator.MyGenerator")
    private String prodId;
 
    // ...
}
```
	In this example, we override the generate() method from the IdentifierGenerator interface and first find the highest number from the existing primary keys of the form prefix-XX.
Then we add 1 to the maximum number found and append the prefix property to obtain the newly generated id value.
Our class also implements the Configurable interface, so that we can set the prefix property value in the configure() method.
Next, let's add this custom generator to an entity. For this, we can use the @GenericGenerator annotation with a strategy parameter that contains the full class name of our generator class:
	
## Composit Identifier
	Composite key or Composit Primary key - group of columns used to uniquely identify 	a record in Db.
	Rules to make a class hold Conposite key:-
	1) it should be defined using @EmbeddedId or @IdClass annotations
	2) it should be public, serializable and have a public no-arg constructor
	3) it should implement equals() and hashCode() methods
```java
		@Embeddable
public class OrderEntryPK implements Serializable {
 
    private long orderId;
    private long productId;
 
    // standard constructor, getters, setters
    // equals() and hashCode() 
}

@Entity
public class OrderEntry {
 
    @EmbeddedId
    private OrderEntryPK entryId;
 
    // ...
}

@Test
public void whenSaveCompositeIdEntity_thenOk() {
    OrderEntryPK entryPK = new OrderEntryPK();
    entryPK.setOrderId(1L);
    entryPK.setProductId(30L);
        
    OrderEntry entry = new OrderEntry();
    entry.setEntryId(entryPK);
    session.save(entry);
 
    assertThat(entry.getEntryId().getOrderId()).isEqualTo(1L);
}	
//Here the OrderEntry object has an OrderEntryPK primary id formed of two attributes: orderId and productId.	
	
```
	
	There is one more way to define Composite primary Key- by using @IdClass annotations:

```java	
public class OrderEntryPK  {

    private long orderId;
    private long productId;
}

@Entity
@IdClass(OrderEntryPK.class)
public class OrderEntry {
    @Id
    private long orderId;
    @Id
    private long productId;
    
    // ...
}

@Test
public void whenSaveIdClassEntity_thenOk() {        
    OrderEntry entry = new OrderEntry();
    entry.setOrderId(1L);
    entry.setProductId(30L);
    session.save(entry);
 
    assertThat(entry.getOrderId()).isEqualTo(1L);
	
```

# Paging and Sorting
	
	##By the help of Pageable Interface and PagingAndSortingRepository by Spring Data JPA
	
	How?
	Pageable<Product> page = new PageRequest(0,1);/// 0 is the first page, 1 is the no of records.
	Page<Product> result = repo.findAll(pageable);
	result.forEach(p-> Sysout(p.getname()))
	
	//it will show 1 record per page.
	
	##Sorting
	** Asc Order**
	repository.findAll(new Sort("name")).forEach(p->System.out.println(p.getName())) ;
	// it will sort in asc order
	
	** DESC Order**
	For desc make changes as- 
	repository.findAll(new Sort(Direction.DESC, "name")).forEach(p->System.out.println(p.getName())) ;
	
	**Sort Using Multiple Props**
	repository.findAll(new Sort(Direction.DESC, "name", "price")).forEach(p->System.out.println(p.getName())) ;
	
	**Sort Using Multiple Props and Directions**
	repository.findAll(new Sort.Order(Direction.DESC, "name"), new Sort.Order("Price") ).forEach(p->System.out.println(p.getName())) ;
	//it will sort byname in Desc order and PRice in ASC order. Order is the Inner class iof Sort class.
	
	** can DO PAging and Sorting in one method?
	Pageable<Product> page = new PageRequest(0,1, Direction.DESC, "name");
	
# Mockito framework and its drawback:
	Mockito is a framework of choice for most of the java based projects. It is easy to implement, read and understand.

	Some of the drawbacks or limitations in terms of functionality are:
	Its inability to mock static methods.
	Constructors, private methods and final classes cannot be mocked.
	
	Solution: Frameworks like PowerMockito (extensions of Mockito framework), JMockit, etc. do provide means to mock private and static methods.

### Test Doubles:
	Used in lieu of External dependency
	DB, Web, API, Network etc

### Mocks:
	It is used for whether function is called or not, if yes, how many times, what parameters are called when it is called?
	In short:
	Right call,
	Right number of times,
	Right number of parameters.
	
### Stub:
	stub checks the behaviour of code if the funtion returns success, failure or error.
	Stub is like making sure a method returns the correct value.
	
	A Mock is just testing behaviour, making sure certain methods are called. A Stub is a testable version (per se) of a particular object.
		