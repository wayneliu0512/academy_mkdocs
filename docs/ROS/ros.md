# ROS 

## Logging 
- log 訊息會紀錄到 `~/.ros/log/xxxx/rosout.log`  
- log 訊息會透過 topic 傳出: `/rosout`([rosgraph_msgs/Log](http://docs.ros.org/en/api/rosgraph_msgs/html/msg/Log.html))
- 內部是採用 log4cxx 的 framework
- ros global 的 log4cxx 參數檔在 `/opt/ros/<ros_dist>/share/ros/config/rosconsole.config`

## Testing 
- 要做 node 之間的測試時, 使用 rostest, 但是如果是對單一 node 內部程式做測試的話, 使用 gtest
- 不能使用 rostest command line tool 去執行 test, 會失敗
- 要使用 下面這些指令  , test 才可以被正常運行, 可以參考 [這個](https://catkin-tools.readthedocs.io/en/latest/verbs/catkin_build.html#building-and-running-tests)

    這用個指令 build:

    ```bash
    $ catkin build <pkg> --catkin-make-args tests
    ```

    用這個指令執行(這其實是 build + execute):

    ```bash
    $ catkin run_tests <pkg>
    ```

- 使用 `$ catkin run_tests` 會有另一個問題, stdout 會產生太多資訊蓋掉 test ouput 結果 可以參考 [這個](https://github.com/catkin/catkin_tools/issues/405)

    ```bash
    $ catkin run_tests <pkg> | sed -En '/^-- run_tests.py/,/^-- run_tests.py/p'
    ```

- test 執行時, log 訊息會失效, 只有 WARN level 以上才會 print 出來, /rosout 則全部都不會被 publish 參考[這個](https://answers.ros.org/question/350204/do-rostest-test-nodes-publish-to-rosout/)
- 若要執行 gtest , 可以直接執行 `.../devel/.private/<pkg>/lib/<pkg>/<test_program>`
- [What is Mocking in Unit test](https://stackoverflow.com/questions/3622455/what-is-the-purpose-of-mock-objects)