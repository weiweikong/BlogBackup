title: my first
date: 2015-10-05 17:38:19
categories: 随笔
tags: 随笔
---

## Test

```
#pragma once

#include <test_mavros/utils/pid_controller.h>

namespace testsetup {
class TestSetup {
public:
	TestSetup() :
		nh("~")
	{ };
	~TestSetup() {};

	ros::NodeHandle nh;

	bool use_pid;
	double rate;
	int num_of_tests;

	void setup(const ros::NodeHandle &nh){
		nh.param("use_pid", use_pid, true);
		nh.param("rate", rate, 10.0);
		nh.param("num_of_tests", num_of_tests, 10);
	}
};
};	// namespace testsetup
```

## Test2

## Test3