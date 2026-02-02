# 🌳 DFS, BFS और BST – Interview Ready Notes (Hindi)

## 1️⃣ DFS (Depth First Search)

### DFS में क्या चाहिए:
- एक **root node**
- एक **stack / recursion**
- एक **values array**

### DFS Iterative (Story Flow):
- root को stack में डालो
- जब तक stack खाली न हो:
  - top निकालो
  - value store करो
  - पहले right, फिर left stack में डालो (LIFO)

```js
function dfs(root) {
  if (!root) return [];
  const stack = [root];
  const values = [];
  while (stack.length) {
    const node = stack.pop();
    values.push(node.key);
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
  return values;
}
```

### DFS Recursive (Preorder)
```js
function dfsRecursive(root) {
  if (!root) return [];
  return [
    root.key,
    ...dfsRecursive(root.left),
    ...dfsRecursive(root.right)
  ];
}
```

---

## 2️⃣ BFS / Level Order Traversal

### BFS क्या है?
- Tree को **level by level** traverse करना
- **Queue (FIFO)** का use

```js
function bfs(root) {
  if (!root) return [];
  const queue = [root];
  const values = [];
  while (queue.length) {
    const node = queue.shift();
    values.push(node.key);
    if (node.left) queue.push(node.left);
    if (node.right) queue.push(node.right);
  }
  return values;
}
```

---

## 3️⃣ Binary Search Tree (BST)

### BST Rule:
- Left < Root < Right

```js
class BstNode {
  constructor(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null;
  }
}
```

### Insert in BST
```js
insert(key) {
  const newNode = new BstNode(key);
  if (!this.root) this.root = newNode;
  else this.insertNode(this.root, newNode);
}

insertNode(node, newNode) {
  if (newNode.key < node.key) {
    if (!node.left) node.left = newNode;
    else this.insertNode(node.left, newNode);
  } else {
    if (!node.right) node.right = newNode;
    else this.insertNode(node.right, newNode);
  }
}
```

### Search
```js
search(value, node = this.root) {
  if (!node) return false;
  if (value === node.key) return true;
  return value < node.key
    ? this.search(value, node.left)
    : this.search(value, node.right);
}
```

### Traversals
```js
inOrderTraversal(node = this.root, res = []) {
  if (node) {
    this.inOrderTraversal(node.left, res);
    res.push(node.key);
    this.inOrderTraversal(node.right, res);
  }
  return res;
}

preOrderTraversal(node = this.root, res = []) {
  if (node) {
    res.push(node.key);
    this.preOrderTraversal(node.left, res);
    this.preOrderTraversal(node.right, res);
  }
  return res;
}

postOrderTraversal(node = this.root, res = []) {
  if (node) {
    this.postOrderTraversal(node.left, res);
    this.postOrderTraversal(node.right, res);
    res.push(node.key);
  }
  return res;
}
```

### Valid BST
```js
isValidBST(node = this.root, min = -Infinity, max = Infinity) {
  if (!node) return true;
  if (node.key <= min || node.key >= max) return false;
  return (
    this.isValidBST(node.left, min, node.key) &&
    this.isValidBST(node.right, node.key, max)
  );
}
```

---

## 🧠 Interview One-Liners
- DFS → stack / recursion
- BFS → queue / level order
- BST inorder → sorted output
