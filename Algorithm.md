## `PriorityTree`

```typescript
class MinHeap {
    data: number[]

    constructor(data = []) {
        this.data = data
        this.heapify()
    }

    heapify() {
        if(this.data.length < 2)
            return
        for(let i = 1; i < this.size(); i++)
            this.bubbleUp(i)
    }

    peek(): number {
        if(this.size() === 0)
            return null
        return this.data[0]
    }

    offer(val: number): void {
        this.data.push(val)
        this.bubbleUp(this.size() - 1)
    }

    poll(): number {
        if(!this.size())
            return null
        const result = this.data[0]
        const last = this.data.pop()
        if(this.size()) {
            this.data[0] = last
            this.bubbleDown(0)
        }
        return result
    }

    bubbleUp(index: number): void {
        while(index) {
            const parentIndex = (index - 1) >> 1
            if(this.data[index] < this.data[parentIndex]) {
                this.swap(index, parentIndex)
                index = parentIndex
            } else
                break
        }
    }

    bubbleDown(index: number): void {
        const lastIndex = this.size() - 1
        while(true){
            const leftChild = index << 1 | 1,
                rightChild = (index << 1) + 2
            let findIndex = index
            if(lastIndex >= leftChild && this.compare(this.data[findIndex], this.data[leftChild]) >= 0)
                findIndex = leftChild
            if(lastIndex >= rightChild && this.compare(this.data[findIndex], this.data[rightChild]) >= 0)
                findIndex = rightChild
            if(index !== findIndex) {
                this.swap(findIndex, index)
                index = findIndex
            } else
                break
        }
    }

    swap(index1: number, index2: number): void {
        [this.data[index1], this.data[index2]] = [this.data[index2], this.data[index1]]
    }

    size(): number {
        return this.data.length
    }

    private compare(a: number, b: number): number {
        return a - b
    }
}
```

