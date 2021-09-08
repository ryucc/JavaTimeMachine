# JavaTimeMachine

## Introduction

Dependencies may cause uncertainties when running our code. We may not own the dependency code or services, and sometimes cannot control their outputs. This is annoying when some random return value causes our code to break. It is a further burden when we make a code fix on a rare edge case, and want to verify the fix.

The JavaTimeMachine is a solution to eliminate this uncertainty. It will run in two modes:
1. **Record mode**: JavaTimeMachine intercepts all dependency calls, and write the inputs and outputs into a run record.
1. **Replay mode**: JavaTimeMachine intercepts all dependency calls, and mocks the dependency behavior using the record file.

Here are some use cases allowed by the JavaTimeMachine:
1. Adding log lines then rerunning the code.
1. Using debuggers.
1. Verifying code fixes on the same input.

It is named "JavaTimeMachine" because it feels like we went back in time to change the code before it was ran.

## Proposed Implementation
### Run modes
Support controlling the run modes by environment variables.

``java MyApplication --JTM_RUN_MODE=[RECORD/REPLAY] --JTM_RECORD_FILE_PATH=[record_file_path]``

Further features may integrate with dependency injection frameworks. e.g. Dagger, Guice, or Spring.

### Programming interface
[TBD] We want to make use of aspect oriented programming.
```java
class MyClass {
}
```
## Additional Features
### Replay options
1. For each dependency, configure if we want to use replay, or call the real service.
2. Support renaming class names.
3. Options for input matching: Match all inputs/Match partial inputs/Match call order.
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

## Challenges
1. System calls
2. Wall clock
3. Crashes
