# Tasks
A **task** is a collection of activities. The activities are arranged in a stack call **back stack**.

多窗口模式下，每个窗口有自己的task。task是一个后入先出的栈结构。task里面的activities是不会重新排列的，只能按顺序进出。

# Managing Tasks
To change the default behavior of tasks, we may use the following `<activity` attributes:
- taskAffinity
- allowTaskReparenting
- clearTaskOnLaunch
- alwaysRetainTaskState
- finishOnTaskLaunch
## Launch Mode
To define the launch mode, we may use the manifest file. And also using intent flags to assist the definition of the launch mode. the priorty of these two are:
1. intent flags
2. manifest
### Manifest
the `launchMode` attribute specifies an instruction on how the activity should be launched into a task.
- standard
- singleTop: If an instance of the activity already exists at the top of the current task, the system routes the intent to that instance through a call to its `onNewIntent()` method, rather than creating a new instance of the activity.可能同时存在多个实例。
- singleTask: The system creates a new task and instantiates the activity at the root of the new task. However, if an instance of the activity already exists in a separate task, the system routes the intent to the existing instance through a call to its `onNewIntent()` method, rather than creating a new instance. So only one instance of the activity can exist at a time. 要不就没实例，要不就在某个task的栈底，不会同时存在多个实例。
- singleInstance: Same as `singleTask`, except that the system doesn't launch any other activites into the task holding the instance. The activity is always the single and only member of its task; any activities started by this one open in a separate task. 在singleTask的基础上更进一步，整个task只有他自己一个实例。不会同时存在多个实例。
### Intent Flags
- FLAG_ACTIVITY_NEW_TASK: This produces the same behavior as the `singleTask` launchMode.
- FLAG_ACTIVITY_SINGLE_TOP: This produces the same behavior as the `singleTop` launchMode.
- FLAG_ACTIVITY_CLEAR_TOP: 如果目标Activity已在当前栈中时，清除掉上面的activity，目标activity根据不同情况决定是否销毁。如果启动模式是默认的standard，并且同一个intent中没有设置FLAG_ACTIVITY_SINGLE_TOP标志，则目标Activity会销毁掉重新创建，因为standard模式下总会创建新的activity实例。如果是其他所有启动模式，或者设置了FLAG_ACTIVITY_SINGLE_TOP，则这个intent直接传递到onNewIntent()方法中。


