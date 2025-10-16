---
title: 渣男和渣函数
categories: Tech
tags: engineering
date: 2025-10-16
---

渣男至少从表面看不像是坏人，但从里面看，他们坏极了。渣函数也一样。

## 关系混乱，处处留情

渣男同时跟多人纠缠不清，关系网复杂混乱。渣函数也一样。

```java
public class PlayerFunction {
    // 到处留情：依赖了数据库、缓存、消息队列、文件系统、第三方服务...
    private UserDao userDao;
    private RedisCache redisCache;
    private MessageQueue mq;
    private FileService fileService;
    private PaymentGateway paymentGateway;
    private EmailService emailService;
    private SmsService smsService;

    public void processUserOrder(int userId, int orderId) {
        // 和数据库搞暧昧
        User user = userDao.findById(userId);
        Order order = orderDao.findById(orderId);

        // 和缓存纠缠不清
        redisCache.set("user:" + userId, user, 3600);

        // 和消息队列有一腿
        mq.send("order_processed", order);

        // 还跟文件系统不清不楚
        fileService.generateReceipt(order);

        // 同时勾搭多个第三方服务
        paymentGateway.charge(order.getAmount());
        emailService.sendReceipt(user.getEmail(), order);
        smsService.sendNotification(user.getPhone(), "订单完成");

        // 这个函数跟谁都有一手，关系网极其复杂
    }
}
```

## 承诺不兑现，善于欺骗

渣男说一套做一套，总是让人失望。渣函数也一样。

```java
public class LiarFunction {
    // 欺骗1：说好只是验证，却偷偷修改数据
    public boolean validateAndProcess(Order order) {
        if (order.isValid()) {
            order.setStatus(Status.PAID); // 说好只是验证，却偷偷修改订单状态
            orderDao.update(order);       // 还更新了数据库！
            return true;
        }
        return false;
    }

    // 欺骗2：承诺返回List，却可能返回null
    public List<User> findActiveUsers() {
        if (someCondition) {
            return userDao.findActive(); // 正常情况
        } else {
            return null; // 特殊情况就违背承诺！
        }
    }

    // 欺骗3：文档说线程安全，实际根本不是
    /**
     * 线程安全的计数器
     */
    public void incrementCounter() {
        count++; // 根本不是线程安全的！
    }
}
```

## 极度自私，不考虑他人

渣男只在乎自己爽不爽，不管别人感受。渣函数也一样。

```java
public class SelfishFunction {
    // 自私1：随意修改传入的参数
    public void calculateDiscount(List<Order> orders) {
        for (Order order : orders) {
            order.setPrice(order.getPrice() * 0.8); // 直接修改原对象
        }
        // 调用者：我的订单列表怎么被改了？！
    }

    // 自私2：占用大量资源不释放
    public void processBigData() {
        Connection conn = getConnection();
        List<Data> hugeList = loadAllData(); // 加载全部数据到内存
        // ... 处理很长时间
        // 忘记关闭连接和释放内存！
    }

    // 自私3：修改全局状态影响其他函数
    public void updateGlobalConfig() {
        System.setProperty("app.mode", "debug"); // 随意修改全局配置
        GlobalSettings.timeout = 0; // 让其他功能都超时！
    }
}
```

## 不负责任，出了问题就甩锅

渣男遇到问题就逃避，从不承担责任。渣函数也一样。

```java
public class IrresponsibleFunction {
    // 甩锅1：对输入完全不检查
    public User getUserById(String id) {
        // 如果id是null、空字符串、或者不存在的ID？
        // 不关我的事，让调用者自己去处理！
        return userMap.get(id);
    }

    // 甩锅2：捕获异常后什么都不做
    public void saveUserData(User user) {
        try {
            userDao.save(user);
            fileService.backup(user);
        } catch (Exception e) {
            // 出了异常？假装没看见！
            // 数据丢失？不是我的问题！
        }
    }

    // 甩锅3：把检查责任推给调用者
    public void transferMoney(Account from, Account to, BigDecimal amount) {
        // 余额不足？账户冻结？金额为负？
        // 这些都是调用者该检查的，我不管！
        from.withdraw(amount);
        to.deposit(amount);
    }
}
```

## 边界感模糊，过度索取

渣男什么都想要，没有分寸感。渣函数也一样。

```java
// 渣函数：边界感模糊过度索取
public class BoundaryLessFunction {
    // 过度索取：需要20个参数！
    public void createUserProfile(
        String username, String password, String email, String phone,
        String firstName, String lastName, Date birthday, String address,
        String city, String country, String zipCode, String gender,
        String occupation, String company, String bio, String avatarUrl,
        boolean newsletter, boolean smsNotification, String referralCode,
        Map<String, Object> preferences) {
        // 这个函数什么都要，没有边界感！
    }

    // 职责过多：一个函数做所有事情
    public void handleUserRegistration(HttpRequest request) {
        // 解析请求
        User user = parseUser(request);

        // 验证数据
        validateUser(user);

        // 保存到数据库
        userDao.save(user);

        // 发送欢迎邮件
        emailService.sendWelcome(user);

        // 创建用户目录
        fileService.createUserFolder(user);

        // 生成API密钥
        String apiKey = generateApiKey(user);

        // 记录日志
        auditService.logRegistration(user);

        // 更新统计
        statsService.incrementUserCount();

        // 这个函数包揽一切，完全没有边界！
    }
}
```

## 情绪不稳定，难以预测

渣男喜怒无常，让人捉摸不透。渣函数也一样。

```java
public class UnpredictableFunction {
    private static Random random = new Random();
    private static boolean systemBusy = false;

    // 难以预测1：依赖随机数
    public boolean shouldProcessRequest() {
        return random.nextDouble() < 0.7; // 70%概率处理？谁知道的！
    }

    // 难以预测2：依赖全局状态
    public String getResponse() {
        if (systemBusy) {
            return "系统繁忙";
        } else {
            return "处理成功";
        }
        // 同样的输入，不同时间返回不同结果！
    }

    // 难以预测3：依赖系统时间
    public boolean isDiscountAvailable() {
        Calendar cal = Calendar.getInstance();
        int hour = cal.get(Calendar.HOUR_OF_DAY);
        return hour % 2 == 0; // 偶数小时有折扣？完全看心情！
    }

    // 难以预测4：有隐藏的竞态条件
    public void updateCounter() {
        if (counter < MAX_VALUE) {
            // 在多线程环境下，这里可能出现竞态条件
            counter++;
        }
    }
}
```

## 情感冷暴力，明知故犯

渣男故意不回应，让人在猜测中痛苦。渣函数也一样。

```java
public class EmotionalAbuseFunction {
    // 冷暴力1：明知是null，还坦然返回
    public User findUserById(int id) {
        if (!userMap.containsKey(id)) {
            // 我知道调用者期待一个User对象
            // 我知道null会导致调用者NPE
            // 但我就是不说，让你自己去猜
            return null;
        }
        return userMap.get(id);
    }

    // 冷暴力2：延迟很久才返回null
    public Data loadHeavyData(String key) {
        try {
            Thread.sleep(5000); // 让调用者苦等5秒
            if (!dataExists(key)) {
                return null; // 最后等来的却是null
            }
            return realData;
        } catch (InterruptedException e) {
            return null; // 连异常都冷处理
        }
    }
}
```

## 怎样才纯情靠谱

一个优秀的男朋友和一个优秀的函数应该怎么做？

1. 专一：一个方法只做好一件事，边界清晰。
2. 诚实：方法签名就是它行为的全部承诺。
3. 负责：要么成功返回值，要么通过 Optional 表面可能为空，要么抛出异常。
4. 独立：通过依赖注入和面向接口编程，易于迁移和测试。
5. 可靠：尊重输入，避免副作用。

想想你有没有在代码库里见过渣函数，以后遇到渣函数你会怎么做？
