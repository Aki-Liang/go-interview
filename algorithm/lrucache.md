# LRU cache

设计一个本地缓存，key最大数量100，淘汰机制LRU

type LRUCache struct {
	size     int
	capacity int
	index    map[int]*CacheNode
	head     *CacheNode
	tail     *CacheNode
}

type CacheNode struct {
	key   int
	value int
	prev  *CacheNode
	next  *CacheNode
}

func Constructor(capacity int) LRUCache {
	head := &CacheNode{}
	tail := &CacheNode{
		prev: head,
	}
	head.next = tail
	c := LRUCache{
		capacity: capacity,
		head:     head,
		tail:     tail,
		index:    make(map[int]*CacheNode),
	}
	return c
}

func (this *LRUCache) Get(key int) int {
	node, ok := this.index[key]
	if !ok {
		return -1
	}
	this.move2Head(node)
	return node.value
}

func (this *LRUCache) Put(key int, value int) {
	node, ok := this.index[key]
	if !ok {
		//add
		node := &CacheNode{
			key:   key,
			value: value,
		}
		this.index[key] = node
		this.add2Head(node)
		this.size++
		if this.size > this.capacity {
			this.removeTail()
		}
	} else {
		node.value = value
		this.move2Head(node)
	}
}

func (this *LRUCache) move2Head(node *CacheNode) {
	this.removeNode(node)
	this.add2Head(node)
}

func (this *LRUCache) add2Head(node *CacheNode) {
	node.next = this.head.next
	node.prev = this.head
	node.next.prev = node
	this.head.next = node
}

func (this *LRUCache) removeNode(node *CacheNode) {
	node.next.prev = node.prev
	node.prev.next = node.next
}

func (this *LRUCache) removeTail() {
	last := this.tail.prev
	this.removeNode(last)
	delete(this.index, last.key)
	this.size--
}
