# User Guide

!!! note
    This guide is for end users, not administrators. If you need help on the deployment process, please go to
    [Deployment Guide](../dev-guide/deployment-guide.md).

aiVLE is short for AI Virtual Learning Environment. It is a platform for evaluating the performance of reinforcement
learning algorithms. Since Python + OpenAI Gym is the most popular toolkit for both writing reinforcement learning
environments and agents, aiVLE provides easy-to-use wrappers on top of existing OpenAI Gym environments and compatible
agents for seamless migration from local/adhoc evaluation scripts to a online/reusable *aiVLE Task*. 

To prepare an *aiVLE Task*, generally you need to provide two parts:

- (Optional) an aiVLE Gym environment: follow [Creating Your Environment](aivle-gym.md) to convert an existing Gym  
- An aiVLE Grader test suite: see [Creating Your Test Suite](aivle-grader.md)

To validate your *aiVLE Task* before submitting to the platform, you may also want to take a look at [Validating 
Sandbox](aivle-worker.md) to have a test-run of your entire package.