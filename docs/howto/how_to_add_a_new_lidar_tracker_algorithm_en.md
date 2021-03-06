# How to add a new lidar tracker algorithm

The default tracking algorithm of Apollo is MlfEngine，which cloud be easily changed or replaced by other algorithms. This document will introduce how to add a new lidar tracker algorithm, the basic task sequence is listed below：

1. Define a class that inherits `base_multi_target_tracker` 
2. Implement the class `NewLidarTracker`
3. Add config and param proto file for `NewLidarTracker`
4. Update lidar_obstacle_tracking.conf

The steps are elaborated below for better understanding:

## Define a class that inherits `base_multi_target_tracker` 

All the lidar tracker algorithms shall inherit `base_multi_target_tracker`，which defines a set of interfaces. Here is an example of a tracker implementation:

```c++
namespace apollo {
namespace perception {
namespace lidar {

class NewLidarTracker : public BaseMultiTargetTracker {
 public:
  NewLidarTracker();
  virtual ~NewLidarTracker() = default;

  bool Init(const MultiTargetTrackerInitOptions& options =
                        MultiTargetTrackerInitOptions()) override;

  bool Track(const MultiTargetTrackerOptions& options, LidarFrame* frame) override;

  std::string Name() const override;

};  // class NewLidarTracker

}  // namespace lidar
}  // namespace perception
}  // namespace apollo
```

## Implement the class `NewLidarTracker`

To ensure the new tracker could function properly, `NewLidarTracker` should at least override the interface Init(), Track(), Name() defined in `base_multi_target_tracker`. Init() is resposible for config loading, class member initialization, etc. And Track() will implement the basic logic of algorithm. A concrete `NewLidarTracker.cc` example is shown：

```c++
namespace apollo {
namespace perception {
namespace lidar {

bool NewLidarTracker::Init(const MultiTargetTrackerInitOptions& options) {
    /*
    Initialization of your tracker
    */
}

bool NewLidarTracker::Track(const MultiTargetTrackerOptions& options, LidarFrame* frame) {
    /*
    Implementation of your tracker
    */
}

std::string NewLidarTracker::Name() const {
    /*
    return your tracker's name
    */
}

PERCEPTION_REGISTER_MULTITARGET_TRACKER(MlfEngine); //register the new tracker

}  // namespace lidar
}  // namespace perception
}  // namespace apollo
```


## Add config and param proto file for `NewLidarTracker`

Follow the following steps to add config and param proto file for the new tracker:

1. Define a `proto` for the new tracker configurations according to the requirements of your algorithm. As a reference， you can found and follow the `proto` definition of `multi_lidar_fusion` at `modules/perception/lidar/lib/tracker/multi_lidar_fusion/proto/multi_lidar_fustion_config.proto`

2. Once finishing your `proto`, for example `newlidartracker_config.proto`, add the following content:

    ```protobuf
    syntax = "proto2";
    package apollo.perception.lidar;

    message NewLidarTrackerConfig {
        double parameter1 = 1;
        int32 parameter2 = 2;
    }
    ```

3. Refer to `modules/perception/production/conf/perception/lidar/config_manager.config` and add your tracker path:

    ```protobuf
    model_config_path: "./conf/perception/lidar/modules/newlidartracker_config.config"
    ```

4. Refer to the `newlidartracker.config` in the same folder and create `modules/multi_lidar_fusion.config`:

    ```protobuf
    model_configs {
    name: "NewLidarTracker"
        version: "1.0.0"
        string_params {
            name: "root_path"
            value: "./data/perception/lidar/models/newlidartracker"
        }
    }
    ```

5. Refer to `multi_lidar_tracker` and create `newlidartracker` folder at `modules/perception/production/data/perception/lidar/models/`. Add `.conf` files for different sensors：

    ```
    Note：The "*.conf" file should have the same structure with the "proto" file defined in step 1，2.
    ```

## Update lidar_obstacle_tracking.conf

To use your new lidar tracker algorithm in Apollo，you need to modify the value of `multi_target_tracker` to your tracker's name in `lidar_obstacle_tracking.conf` located in corresponding sensor folder in `modules/perception/production/data/perception/lidar/models/lidar_obstacle_pipline`

Once you finished the above modifications, you new tracker should take effect in Apollo.
