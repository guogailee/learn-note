## Spring Cache特点

- 通过少量的配置annotation注解即可使既有代码支持缓存。
- 支持开箱即用out-of-the-box，即不用安装和部署额外第三方组件即可使用缓存。
- 支持Spring Express Language，能使用对象的任何属性或者方法来定义缓存的key和condition。
- 支持 AspectJ，并通过其实现任何方法的缓存支持。
- 支持自定义 key 和自定义缓存管理者，具有相当的灵活性和扩展性

## 不使用Spring Cache

```
public class AccountService {
    private MyCacheManager<Account> cacheManager;

    public AccountService(MyCacheManager<Account> cacheManager) {
        this.cacheManager = cacheManager;
    }

    public Account queryAccountByName(String name) {
        Account account = cacheManager.getValue(name);
        if (account != null) {
            System.out.println("query from cache");
            return account;
        }
        account = getFromDB(name);
        if (account != null) {
            cacheManager.addOrUpdateValue(name, account);
        }
        return account;

    }

    public Account getFromDB(String name) {
        System.out.println("query from db");
        return new Account(name);
    }

    public void reload() {
        cacheManager.evictCache();
    }

}
```

## 使用Spring Cache

```
public class AccountService {


    @Cacheable(value = "accountCache", condition = "#userName.length()<=4")
    public Account getAccountByName(String userName) {
        return getAccountFromDB(userName);
    }

    @Cacheable(value = "accountCache", key = "#userName.concat(#id)")
    public Account getAccountByName2(String userName, int id) {
        return getAccountFromDB(userName);
    }

    @CacheEvict(value = "accountCache", key = "#account.getName()")
    public void updateAccount(Account account) {
        updateDB(account);
    }

    @CachePut(value = "accountCache", key = "#account.getName()")
    public void updateAccount2(Account account) {
        updateDB(account);
    }

    @CacheEvict(value = "accountCache", allEntries = true)
    public void reload() {

    }

    public Account getAccountFromDB(String userName) {
        System.out.println("query from db");
        return new Account(userName);
    }

    private void updateDB(Account account) {
        System.out.println("update to db");
    }
}
```

- @Cacheable 主要针对方法配置，能够根据方法的请求参数对结果进行缓存

  value key condition

- @CacheEvict 主要针对方法配置，能够根据一定的条件对缓存进行清空

  value key condition allEntries beforeInvocation 

- 基本原理：spring aop

- 扩展：可以替换为任何的缓存框架

- 可以结合spring表达式语言，灵活使用