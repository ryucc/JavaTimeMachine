# JavaTimeMachine

## Introduction

Dependencies may cause uncertainties when running our code. We may not own the dependency code or services, and sometimes cannot control their outputs. This is annoying when some random return value causes our code to break. It is a further burden when we make a code fix on a rare edge case, and want to verify the fix.

The JavaTimeMachine is a solution to eliminate this uncertainty. It will run in two modes:
1. **Record mode**: JavaTimeMachine intercepts and records all dependency calls, writing the inputs and outputs into a run record.
1. **Replay mode**: JavaTimeMachine intercepts and mocks all dependency calls, using the record file as the data source.

Here are some use cases allowed by the JavaTimeMachine:
1. Rerunning after adding patches. e.g. add a log line, implement a code fix.
1. Using debuggers to find errors.

It is named "JavaTimeMachine" because it feels like we went back in time to change the code before it was ran.

## Proposed Implementation
### Run modes
Support controlling the run modes by environment variables.

``java MyApplication --JTM_RUN_MODE=[RECORD/REPLAY] --JTM_RECORD_FILE_PATH=[record_file_path]``

### Programming interface
[TBD] We want to make use of aspect oriented programming. In the vanilla version, we define the `@DependencyCall` annotation. Developers can imagine the equivalent code: 
```java
class MyDependency {
  @DependencyCall
  public Output queryObjects(Input key){
    ...
    return retVal;
  }
}
```

Translates to:
```java
class MyDependency {
  @DependencyCall
  public Output queryObjects(Input key){
    if (JavaTimeMachine.isRecordMode()) {
      JavaTimeMachine.recordInput(key);
      try {
        ...
        JavaTimeMachine.recordOutput(retVal);
        return retVal;
      } catch (Exception e) {
        JavaTimeMachine.recordException(e);
        throw e;
      }
    } else {
      return JavaTimeMachine.mockBehavior(key, this.class);
    }
  }
}
```
### Record file specs
Define some required metadata for JTM to run. Additional metadata should not crash JTM. Great if the file is human readable.
1. Class name & method signatures
1. Input/output in portable format. (XML, JSON, ...etc)
1. [TBD] Call order
1. [TBD] Thread id
1. [TBD] Object identifier, incase there are multiple instances of same dependency.

Example
```
dependencyCall: Output com.demo.MyDependency$queryObjects(Input)
  callOrder: 3
  threadId: 0
  Input: {"field1": "abc", "field2": "def"}
  Output: {"field1": "abc", "field2": "def"}

dependencyCall: Output com.demo.MyDependency$queryObjects(Input)
  callOrder: 4
  threadId: 0
  Input: {"field1": "abc", "field2": "def"}
  Exception Thrown: java.io.IOException {"message": "abc"}
...
```

### Version 0 demo implementation & Resrictions
1. Only single thread support.
2. Input/Outputs must be POJOs. Returning an HttpClient will probably crash in rerun.
3. Primitive types in the Input/Output Pojos?
4. [TBD] Use a singleton JavaTimeMachine object.
5. [TBD] Guice AOP, Gson.

## Additional Features
### Replay options
1. Configure to use mock or call real per dependency.
2. Class renaming support.
3. Input matching option: Match exact input/Match partial input/Match by call order and className.
### Recording on methods
When we don't own the main method, we may want to just replay parts of the code. For example, AWS Lambda provides an interface for devs to implement. The main method is hidden, and maybe hard to test locally.

An @JavaTimeMachineScope(TBD) and @JavaTimeMachineMainMethod annotaion can be introducted to solve this problem. This annotation should spy on the dependencies in the constructor, records inputs on the main method. In replay mode, JTM should be able to execute the main method directly, skipping the framework code.

```java
@JavaTimeMachineScope
public class HandlerDynamoDB implements RequestHandler<DynamodbEvent, String>{
  @JavaTimeMachineMainMethod
  public String handleRequest(DynamodbEvent event, Context context)
  {
  ...
  }
}
```
### Cloud Storage Support
Another use case for AWS Lambda. Cloud functions may provide temporary disk storage, but we need the files to be saved. There can be some integration with S3.

### Integration with DI frameworks
Initialize infrastructure and settings with DI frameworks. e.g. Dagger, Guice, or Spring.

## Challenges
1. System calls/hidden dependencies.
2. Wall clock
3. Crashes
