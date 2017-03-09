---
layout: post

title: Xcode单元测试

date: 2017-03-02 15:32:24.000000000 +09:00

---

### 基础逻辑测试
```
    override func setUp() {
        super.setUp() //为开始测试准备环境
        api = RxMoyaProvider<LemeAPI>(stubClosure: MoyaProvider.neverStub)
    }
    override func tearDown() {
        super.tearDown()//测试结束释放资源
    }
    func testFuncName(){//测试方法以test开头
        let ret = FuncName()//return "88"
        XCTAssertEqual(ret, "88")
    }

```
### 异步加载测试
    func testGetSomeData() { //以test开头
        let e = expectation(description: "GetSomeData Error")
        self.api
            .request(LemeAPI.GetSomeData)
            .map(to: SomeData.self)
            .subscribe(onNext: { (someData) -> Void in
                XCTAssertEqual(someData.login, "http://api.spottly.com/login")
                e.fulfill()
            }, onError: { (error) -> Void in
                print(error)
                XCTFail("Error")
            }).addDisposableTo(self.disposeBag)
        
        waitForExpectations(timeout: 10) { (error) in
            print(error)
        }
    }


### 数据Mock测试
当测试的目标依赖其他对象时，可以用Mock用于模拟一个对象的返回
##### MyClass.swift
```
import Foundation
import CoreData

class MyClass {
    let context: NSManagedObjectContext
    
    init(managedObjectContext: NSManagedObjectContext) {
        self.context = managedObjectContext
    }
    
    // If the array returned from executing a fetch request contains objects, return true; if empty, return false
    func databaseHasRecordsForSomeEntity() -> Bool {
        let fetchRequest = NSFetchRequest(entityName: "SomeEntity")
        let fetchRequestResults = self.context.executeFetchRequest(fetchRequest, error: nil) // May want to do something with the error in real life...
        return (fetchRequestResults?.count > 0)
    }
}
```
##### TestFile.swift

```
  // Yay for verbose test names!  :]
    func testDatabaseHasRecordsForSomeEntityReturnsTrueWhenFetchRequestReturnsNonEmptyArray() {
        class MockNSManagedObjectContext: NSManagedObjectContext {
            override func executeFetchRequest(request: NSFetchRequest, error: NSErrorPointer) -> [AnyObject]? {
                return ["object 1"]
            }
        }
        
        // Instantiate mock
        let mockContext = MockNSManagedObjectContext()
        
        // Instantiate class under test and pass it the mockContext object
        let myClassInstance = MyClass(managedObjectContext: mockContext)
        
        // Call the method under test and store its return value for XCTAssert
        let returnValue = myClassInstance.databaseHasRecordsForSomeEntity()
        
        XCTAssertTrue(returnValue == true, "The return value should be been true")
    }
```
[参考](https://www.andrewcbancroft.com/2014/07/15/how-to-create-mocks-and-stubs-in-swift/)
### Code Coverage 测试覆盖率
在product->scheme->Edit Scheme->Test里面将Code Coverage模式打开
测试成功后可以在这里看到
![](/assets/images/WX20170302-155306@2x.png){:  height="200px"}

[参考](http://www.jianshu.com/p/f4ba532caed0)