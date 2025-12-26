```python fold file:demo.py
import asyncio

import multiprocessing as mp

import os

import time

  

NEW_REQUEST = "new_request"

ENGINE_STOP = "engine_stop"

ENGINE_OUTPUT = "engine_output"

ENGINE_SHUTDOWN = "engine_shutdown"

  
  

def engine_core_main(cmd_q, resp_q):

    print("[EngineCore] 启动")

  

    active = {}  # req_id -> step

  

    while True:

        # 1️⃣ 收新请求

        while not cmd_q.empty():

            msg = cmd_q.get()

            if msg["type"] == "ENGINE_STOP":

                print("[EngineCore] 收到 STOP")

                resp_q.put({"type": "ENGINE_SHUTDOWN"})

                return

  

            if msg["type"] == "NEW_REQUEST":

                rid = msg["req_id"]

                active[rid] = 0

                print(f"[EngineCore] 注册请求 {rid}")

  

        # 2️⃣ decode step

        finished = []

        for rid in active:

            active[rid] += 1

            step = active[rid]

  

            resp_q.put(

                {

                    "type": "ENGINE_OUTPUT",

                    "req_id": rid,

                    "token": f"token_{step}",

                    "finished": step >= 3,

                }

            )

  

            if step >= 3:

                finished.append(rid)

  

        for rid in finished:

            del active[rid]

  

        time.sleep(0.5)

  
  

class EngineClient:

    def __init__(self, cmd_q, resp_q, loop):

        self.cmd_q = cmd_q

        self.resp_q = resp_q

        self.loop = loop

        self.futures = {}

  

    async def submit(self, prompt):

        req_id = id(prompt)

        fut = self.loop.create_future()

        self.futures[req_id] = fut

  

        self.cmd_q.put(

            {

                "type": "NEW_REQUEST",

                "req_id": req_id,

                "prompt": prompt,

            }

        )

  

        return await fut

  

    async def response_loop(self):

        while True:

            msg = await asyncio.to_thread(self.resp_q.get)

  

            if msg["type"] == "ENGINE_SHUTDOWN":

                print("[Client] Engine 已关闭")

                break

  

            if msg["type"] == "ENGINE_OUTPUT":

                rid = msg["req_id"]

                print(f"[Client] {rid} <- {msg['token']}")

  

                if msg["finished"]:

                    fut = self.futures.pop(rid, None)

                    if fut:

                        fut.set_result(f"结果完成 ({rid})")

  

async def main():

    cmd_q = mp.Queue()

    resp_q = mp.Queue()

  

    p = mp.Process(target=engine_core_main, args=(cmd_q, resp_q))

    p.start()

  

    loop = asyncio.get_running_loop()

    client = EngineClient(cmd_q, resp_q, loop)

  

    listener = asyncio.create_task(client.response_loop())

  

    results = await asyncio.gather(

        client.submit("你好"),

        client.submit("天气不错"),

    )

  

    for r in results:

        print("[Main]", r)

  

    cmd_q.put({"type": "ENGINE_STOP"})

    await listener

    p.join()

  

if "__main__" == __name__:

    asyncio.run(main())
```
 
```python fold file:zmq.py

import asyncio  

import multiprocessing as mp  

import uuid  

import zmq  

import zmq.asyncio  

import msgspec.msgpack  

# 消息类型  

NEW_REQUEST = "new_request"  

ENGINE_STOP = "engine_stop"  

ENGINE_OUTPUT = "engine_output"  

ENGINE_SHUTDOWN = "engine_shutdown"  

def engine_core_main(input_addr, output_addr):  

    """使用ZMQ的engine core"""  

    ctx = zmq.Context()  

    # DEALER socket接收请求  

    input_socket = ctx.socket(zmq.DEALER)  

    input_socket.connect(input_addr)  

    # PUSH socket发送响应  

    output_socket = ctx.socket(zmq.PUSH)  

    output_socket.connect(output_addr)  

    # 发送初始握手消息  

    input_socket.send(b"")  

    active = {}  # req_id -> step  

    poller = zmq.Poller()  

    poller.register(input_socket, zmq.POLLIN)  

    try:  

        while True:  

            # 等待请求或超时处理  

            socks = dict(poller.poll(timeout=500))  

            if input_socket in socks:  

                # 接收请求  

                msg_type, *data_frames = input_socket.recv_multipart()  

                msg_type = msg_type.decode()  

                if msg_type == ENGINE_STOP:  

                    output_socket.send_multipart([  

                        ENGINE_SHUTDOWN.encode()  

                    ])  

                    break  

                elif msg_type == NEW_REQUEST:  

                    # 解析请求  

                    request = msgspec.msgpack.decode(data_frames[0])  

                    req_id = request["req_id"]  

                    active[req_id] = 0  

                    print(f"[EngineCore] 注册请求 {req_id}")  

            # 处理活跃请求  

            finished = []  

            for rid in list(active.keys()):  

                active[rid] += 1  

                step = active[rid]  

                response = {  

                    "type": ENGINE_OUTPUT,  

                    "req_id": rid,  

                    "token": f"token_{step}",  

                    "finished": step >= 3,  

                }  

                output_socket.send_multipart([  

                    msgspec.msgpack.encode(response)  

                ])  

                if step >= 3:  

                    finished.append(rid)  

            for rid in finished:  

                del active[rid]  

    finally:  

        input_socket.close()  

        output_socket.close()  

        ctx.term()  

class ZMQEngineClient:  

    """使用ZMQ的engine客户端"""  

    def __init__(self, input_addr, output_addr):  

        self.ctx = zmq.asyncio.Context()  

        # ROUTER socket发送请求  

        self.input_socket = self.ctx.socket(zmq.ROUTER)  

        self.input_socket.bind(input_addr)  

        # PULL socket接收响应  

        self.output_socket = self.ctx.socket(zmq.PULL)  

        self.output_socket.bind(output_addr)  

        self.futures = {}  

        self.output_queue = asyncio.Queue()  

    async def submit(self, prompt):  

        """提交请求"""  

        req_id = str(uuid.uuid4())  

        fut = asyncio.get_running_loop().create_future()  

        self.futures[req_id] = fut  

        request = {  

            "req_id": req_id,  

            "prompt": prompt,  

        }  

        # 发送到engine (identity为空字符串)  

        await self.input_socket.send_multipart([  

            b"",  # 空identity  

            NEW_REQUEST.encode(),  

            msgspec.msgpack.encode(request)  

        ])  

        return await fut  

    async def response_loop(self):  

        """处理响应"""  

        while True:  

            try:  

                frames = await self.output_socket.recv_multipart()  

                response = msgspec.msgpack.decode(frames[0])  

                if response["type"] == ENGINE_SHUTDOWN:  

                    print("[Client] Engine 已关闭")  

                    break  

                elif response["type"] == ENGINE_OUTPUT:  

                    rid = response["req_id"]  

                    print(f"[Client] {rid} <- {response['token']}")  

                    if response["finished"]:  

                        fut = self.futures.pop(rid, None)  

                        if fut:  

                            fut.set_result(f"结果完成 ({rid})")  

            except Exception as e:  

                print(f"[Client] 错误: {e}")  

                break  

    async def shutdown(self):  

        """关闭客户端"""  

        await self.input_socket.send_multipart([  

            b"", ENGINE_STOP.encode()  

        ])  

        self.input_socket.close()  

        self.output_socket.close()  

        self.ctx.term()  

async def main():  

    # ZMQ地址  

    input_addr = "tcp://127.0.0.1:5555"  

    output_addr = "tcp://127.0.0.1:5556"  

    # 启动engine进程  

    p = mp.Process(  

        target=engine_core_main,  

        args=(input_addr, output_addr)  

    )  

    p.start()  

    # 创建客户端  

    client = ZMQEngineClient(input_addr, output_addr)  

    # 启动响应处理  

    response_task = asyncio.create_task(client.response_loop())  

    # 等待engine就绪  

    await asyncio.sleep(0.1)  

    # 提交请求  

    results = await asyncio.gather(  

        client.submit("你好"),  

        client.submit("天气不错"),  

    )  

    for r in results:  

        print("[Main]", r)  

    # 关闭  

    await client.shutdown()  

    await response_task  

    p.join()  

if __name__ == "__main__":  

    asyncio.run(main())
```
