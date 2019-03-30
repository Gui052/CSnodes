# Java数组实现队列

```Java
public class ArrayQueue {
    private int[] arrayqueue;
    private int front, rear, length, count;
    public ArrayQueue(int leagth) {
        arrayqueue = new int[leagth + 1];
        this.length = leagth + 1;
        front = 0;
        rear = 1;
        count = 0;
    }
    public int getCount() {
        return count;
    }
    public void inqueue(int data) {
        if (rear % length == front) {
            System.out.println("队列已满");
        } else {
            arrayqueue[(rear - 1 + length) % length] = data;
            rear = (rear + 1) % length;
            count++;
        }
    }
    public int dequeue() {
        if ((front + 1) % length == rear) {
            System.out.println("队列空");
            return -1;
        } else {
            int ret = arrayqueue[front];
            front = (front + 1) % length;
            count--;
            return ret;
        }
    }
}
```

