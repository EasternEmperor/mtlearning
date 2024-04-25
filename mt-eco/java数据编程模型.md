
1. PO(Persisited Object)：表示持久化对象，通常与数据库中的表结构对应，包含与数据库字段一致的属性，提供相应的getter和setter方法
    
    1. 通过ORM技术（MyBatis等）与数据库等表进行映射，getter和setter方法对外提供属性的访问和修改等操作
        
2. DO(Domain Object)：表示领域对象，是业务逻辑中的实体对象，通常用于封装业务数据，包含业务领域相关的属性和行为方法
    
    1. 比如User类包含用户数据，也提供更改密码和更新个人资料等业务方法
        
        ```java
        public class UserDO {
            private String userId;
            private String username;
            private String password;
            private String email;
            
            public void changePassword(String newPassword) {
                // 验证密码复杂度
                // 如果通过，更新密码
            }
            
            public void updateProfile(String newEmail) {
                // 验证邮箱格式
                // 如果通过，更新邮箱地址
            }
            
            // 省略其他getter和setter方法
        }
        ```
        
3. TO(Transfer Object)：表示传输对象，用于在远程调用或分布式系统中传输数据，通常比较简单
    
    1. 用于数据传输，封装多个数据字段便于一次性传输多个数据项，减少远程调用的次数，不包含业务逻辑处理
        
        ```java
        public class OrderTO {
            private String orderId;
            private String customerId;
            private List<ItemTO> items;
            private BigDecimal totalAmount;
            
            // 省略getter和setter方法
        }
        ```
        
4. DTO(Data Transfer Object)：与TO类似，用于在层间或模块间传输数据，但包含更多的属性和行为方法，用于封装从数据库查询结果中获取的数据，并进行业务处理
    
    1. 在不同层间传输数据，例如当从数据库中检索客户信息并需要发送到前端显示时，可以使用CustomerDTO来传输这些数据。这样，即使客户的模型在数据库中有更复杂的结构，DTO也可以只包含前端所需要的数据字段。
        
        ```java
        public class CustomerDTO {
            private String id;
            private String name;
            private String email;
            
            // 构造器、getter和setter方法省略
        }
        ```
        
5. VO(View Object)：表示视图对象，用于封装展示给用户的数据，与界面视图一一对应，包含与界面显示相关的属性
    
    1. 在MVC架构中，控制器从模型层获取数据封装到VO中，然后传递给视图层展示
        
        ```java
        public class CustomerVO {
            private String name;
            private String email;
            // 可能包含其他为了展示而定制的字段
            
            // 构造器、getter和setter方法省略
        }
        ```
        
6. BO(Business Object)：表示业务对象，用于封装复杂的业务逻辑处理
    
7. POJO(Plain Old Java Object)：表示普通的Java对象，指没有任何特殊限制和约束的普通Java类
    
8. DAO(Data Access Object)：表示数据访问对象，主要用于封装对数据库的访问操作，包含查询、更新、删除等操作方法
    
    1. 主要目的是将数据持久化层与业务逻辑分离开，提供清晰的数据访问接口。DAO用于对数据库进行CRUD操作
        
        ```java
        public interface CustomerDAO {
            Customer findById(int id);
            List<Customer> findAll();
            void save(Customer customer);
            void update(Customer customer);
            void delete(Customer customer);
        }
        
        public class CustomerDAOImpl implements CustomerDAO {
            // 数据库连接和操作的实现代码省略
            
            @Override
            public Customer findById(int id) {
                // 实现查找客户的逻辑
            }
            
            @Override
            public List<Customer> findAll() {
                // 实现返回所有客户的逻辑
            }
            
            @Override
            public void save(Customer customer) {
                // 实现保存客户的逻辑
            }
            
            @Override
            public void update(Customer customer) {
                // 实现更新客户信息的逻辑
            }
            
            @Override
            public void delete(Customer customer) {
                // 实现删除客户的逻辑
            }
        }
        ```