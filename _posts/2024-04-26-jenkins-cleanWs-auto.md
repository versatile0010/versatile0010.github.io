---
title: " 🧹 How to disable automatic cleanup in jenkins?"

categories: 
  - infra
layout: single
classes: wide
last_modified_at: now
---

> Jenkins automatically cleans up old workspaces using WorkspaceCleanupThread to manage disk space.
This happens periodically, even without the cleanWs() command in your pipeline, ensuring efficient disk space usage.

### Jenkins workspace

![image](https://github.com/versatile0010/versatile0010.github.io/assets/96612168/ccd3eced-b456-4dde-9ebd-27941fb3dcb9)

젠킨스(Jenkins) 에서는 각각의 job build process 마다 고유한 workspace 을 생성하여 사용합니다.

workspace 에는 build 시 필요한 파일들이 저장되며,

여러 번의 실행에도 독립적인 환경에서 build 가 가능하도록 제공해줍니다.

workspace 에 적재되는 파일의 크기가 큰 경우,

jenkins server 의 disk 가 가득 찰 수 있는 문제가 발생할 수 있습니다.

이를 방지하기 위해, 

파이프라인에 `cleanWs()` 명령어를 추가하여 각 build process 마다 workspace 를 지워주는 작업을 선행/후행하도록 설정할 수 있죠.

workspace 를 유지하고 싶다면 `cleanWs()` 를 사용하지 않으면 될 것만 같지만,

실제로는 일정 주기가 지나면 오래된 workspace 는 스스로 제거됩니다.


운영 환경에서 동작하고 있던 jenkins job 중 일부는 파이프라인에 cleanWs() 를 사용하고 있지 않았습니다.

그럼에도 console log 를 보면 clean workspace 를 시도한 것을 확인할 수 있었죠.


왜 이렇게 동작하는 지, 살펴보았습니다.

### WorkspaceCleanupThread

서치해보니 젠킨스 시스템 내에서 주기적으로 workspace 를 cleanup 하여, 젠킨스 서버의 disk 공간을 관리한다고 합니다.

관련 세부 로직은 `WorkspaceCleanupThread.java` 에서 확인할 수 있었습니다.

- [https://github.com/jenkinsci/jenkins/blob/171c4d7f6d6ea41e52e60aa819ea7fc7985209cc/core/src/main/java/hudson/model/WorkspaceCleanupThread.java](https://github.com/jenkinsci/jenkins/blob/171c4d7f6d6ea41e52e60aa819ea7fc7985209cc/core/src/main/java/hudson/model/WorkspaceCleanupThread.java)

주의깊게 확인해야 하는 부분에는 ⭐️이모지를 달아두었습니다. 

```java
/**
 * Clean up old left-over workspaces from agents.
 *
 * @author Kohsuke Kawaguchi
 */
@Extension @Symbol("workspaceCleanup")
public class WorkspaceCleanupThread extends AsyncPeriodicWork {
    // ... 생략 ...
    @Override protected void execute(TaskListener listener) throws InterruptedException, IOException {
        if (disabled) { <------ ⭐️
            LOGGER.fine("Disabled. Skipping execution");
            return;
        }
        // ... 생략 ...
        for (TopLevelItem item : j.allItems(TopLevelItem.class)) {
            if (item instanceof ModifiableTopLevelItemGroup) { // no such thing as TopLevelItemGroup, and ItemGroup offers no access to its type parameter
                continue; // children will typically have their own workspaces as subdirectories; probably no real workspace of its own
            }
            listener.getLogger().println("Checking " + item.getFullDisplayName());
            for (Node node : nodes) {
                FilePath ws = node.getWorkspaceFor(item);
                if (ws == null) {
                    continue; // offline, fine
                }
                boolean check;
                try {
                    check = shouldBeDeleted(item, ws, node); <------ ⭐️
                } catch (IOException | InterruptedException x) {
                    Functions.printStackTrace(x, listener.error("Failed to check " + node.getDisplayName()));
                    continue;
                }
                if (check) { <------ ⭐️
                    listener.getLogger().println("Deleting " + ws + " on " + node.getDisplayName());
                    try {
                        ws.deleteRecursive(); <------ ⭐️
                        WorkspaceList.tempDir(ws).deleteRecursive(); <------ ⭐️
                    } catch (IOException | InterruptedException x) {
                        Functions.printStackTrace(x, listener.error("Failed to delete " + ws + " on " + node.getDisplayName()));
                    }
                }
            }
        }
    }

    private boolean shouldBeDeleted(@Nonnull TopLevelItem item, FilePath dir, @Nonnull Node n) throws IOException, InterruptedException {
        // ... 생략 ...
        if(dir.lastModified() + retainForDays * DAY > now) { <------ ⭐️
            LOGGER.log(Level.FINE, "Directory {0} is only {1} old, so not deleting", new Object[] {dir, Util.getTimeSpanString(now-dir.lastModified())});
            return false;
        }
        // ... 생략 ...
        return true;
    }
    
    /**
     * Can be used to disable workspace clean up. <------ ⭐️
     */
    public static boolean disabled = SystemProperties.getBoolean(WorkspaceCleanupThread.class.getName()+".disabled");

    /**
     * How often the clean up should run. This is final as Jenkins will not reflect changes anyway. <------ ⭐️
     */
    public static final int recurrencePeriodHours = SystemProperties.getInteger(WorkspaceCleanupThread.class.getName()+".recurrencePeriodHours", 24);

    /**
     * Number of days workspaces should be retained. <------ ⭐️
     */
    public static int retainForDays = SystemProperties.getInteger(WorkspaceCleanupThread.class.getName()+".retainForDays", 30);
}

```

위 코드를 통해 알 수 있는 점을 정리해보았습니다.

```java
- recurrencePeriodHours 주기마다 workspace clean up 을 수행 (기본값: 24 시간)
- 삭제해야 할 workspace 를 판단하는 기준은, 마지막 수정 시간이 retainForDays 보다 오래되었는지를 체크 (기본값: 30 일)
```

젠킨스 job 파이프라인에서 cleanWs() 를 사용하지 않더라도, 젠킨스 서버에서 workspace 를 clean up 하려고 한 이유는 이제 분명해졌죠.

만약, WorkspaceCleanupThread 에 의한 workspace clean up 이 일어나지 않도록 방지하려면 어떻게 해야 할까요?

`WorkspaceCleanupThread.java` 코드를 다시 확인해봅시다.

```java

@Override protected void execute(TaskListener listener) throws InterruptedException, IOException {
  if (disabled) { <------ ⭐️
    LOGGER.fine("Disabled. Skipping execution");
    return;
  }
  // ... 생략 ...
}

    /**
     * Can be used to disable workspace clean up. <------ ⭐️
     */
    public static boolean disabled = SystemProperties.getBoolean(WorkspaceCleanupThread.class.getName()+".disabled");

```

`disabled` 값에 따라서 로직이 수행되는 것을 알 수 있고, 해당 값은 환경 변수로 전달받으므로

Jenkins 시작 시, 


`-Dhudson.model.WorkspaceCleanupThread.disable=true` 를 추가하여 workspace clean up 을 비활성화 할 수 있습니다.

---

잘못된 부분이 있다면 comment 남겨주세요!

---

## References
- [https://plugins.jenkins.io/ws-cleanup/](https://plugins.jenkins.io/ws-cleanup/)
- [https://community.jenkins.io/t/some-processor-or-thread-is-cleaning-up-my-workspace/12376](https://community.jenkins.io/t/some-processor-or-thread-is-cleaning-up-my-workspace/12376)
