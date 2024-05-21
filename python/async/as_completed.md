# Optimizing Asynchronous Request Handling in Python with `asyncio.as_completed`

Consider a scenario where you are managing a list of items, each requiring an initial request. Each request could yield another list of results. Consequently, based on the values received from these results, a second set of requests must be dispatched. While `asyncio.gather` is typically employed to execute these asynchronous requests concurrently, it has the drawback of halting the processing of subsequent requests until all initial ones are complete. This is suboptimal because optimally, we would want to initiate the second batch of requests immediately after any of the initial requests is fulfilled.


To illustrate this with an example, suppose we have a list of three items [a, b, c], which generate first-round requests [a1, b1, c1]. For the second round, we might need to send requests [a21, a22, b21, b22, b23, c21]. Using `asyncio.gather`, the second-round requests would be delayed until the entirety of [a1, b1, c1] have been processed. However, our goal is to proceed with the second-round requests for each item as soon as its first request is completed.


To address this, we can utilize `asyncio.as_completed`. This function effectively attaches a completion callback to each future and dispatches the result to a queue. It then yields results from the queue as they become available, allowing for the second round of requests to be initiated promptly after any first-round request concludes. Example code:

```python
import asyncio


async def second_task(num):
    print("second task", num)
    if num > 3:
        await asyncio.sleep(num)
    else:
        await asyncio.sleep(1)
    print("finished second task", num)
    return [num, num + 1]


async def third_task(num):
    print("third task", num)
    await asyncio.sleep(num)
    return num


async def main(loop):
    nums = [5, 1, 2, 3, 4]
    tasks = []
    for num in nums:
        tasks.append(loop.create_task(second_task(num)))

    new_tasks = []
    for task in asyncio.as_completed(tasks):
        try:
            result = await task
            for num in result:
                new_tasks.append(loop.create_task(third_task(num)))
        except Exception as e:
            print(f"Error occurred: {e}")

    for task in asyncio.as_completed(new_tasks):
        try:
            result = await task
            print(f"Result: {result}")
        except Exception as e:
            print(f"Error occurred: {e}")


if __name__ == "__main__":
    loop = asyncio.new_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```