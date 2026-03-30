import java.util.*;

public class UPTMMarketingOptimization {

    static int[][] costMatrix = {
        {0, 15, 25, 35},
        {15, 0, 30, 28},
        {25, 30, 0, 20},
        {35, 28, 20, 0}
    };

    static String[] locations = {"UPTM", "City B", "City C", "City D"};

    // greedy
    public static String greedyMCOP(int[][] dist) {
        boolean[] visited = new boolean[dist.length];
        int cost = 0, current = 0;

        String path = locations[0];
        visited[0] = true;

        for (int i = 0; i < dist.length - 1; i++) {
            int min = Integer.MAX_VALUE, next = -1;

            for (int j = 0; j < dist.length; j++) {
                if (!visited[j] && dist[current][j] < min) {
                    min = dist[current][j];
                    next = j;
                }
            }

            visited[next] = true;
            cost += min;
            current = next;
            path += " -> " + locations[next];
        }

        cost += dist[current][0];
        path += " -> UPTM";

        return "Greedy Route: " + path + " | Total Cost: " + cost;
    }

    // dynamic programming
    public static String dynamicProgrammingMCOP(int[][] dist) {
        int n = dist.length;
        int VISITED_ALL = (1 << n) - 1;

        int[][] memo = new int[n][1 << n];
        String[][] pathMemo = new String[n][1 << n];

        for (int[] row : memo) Arrays.fill(row, -1);

        int cost = tsp(0, 1, dist, memo, pathMemo, VISITED_ALL);
        String path = "UPTM" + pathMemo[0][1] + " -> UPTM";

        return "Dynamic Programming Route: " + path + " | Total Cost: " + cost;
    }

    static int tsp(int pos, int mask, int[][] dist,
                   int[][] memo, String[][] pathMemo, int VISITED_ALL) {

        if (mask == VISITED_ALL)
            return dist[pos][0];

        if (memo[pos][mask] != -1)
            return memo[pos][mask];

        int ans = Integer.MAX_VALUE;
        String bestPath = "";

        for (int city = 0; city < dist.length; city++) {
            if ((mask & (1 << city)) == 0) {

                int newCost = dist[pos][city] +
                        tsp(city, mask | (1 << city), dist, memo, pathMemo, VISITED_ALL);

                if (newCost < ans) {
                    ans = newCost;
                    bestPath = " -> " + locations[city] +
                            (pathMemo[city][mask | (1 << city)] == null ? "" :
                             pathMemo[city][mask | (1 << city)]);
                }
            }
        }

        pathMemo[pos][mask] = bestPath;
        return memo[pos][mask] = ans;
    }

    // backtracking
    static int minCostBT;
    static String bestPathBT;

    public static String backtrackingMCOP(int[][] dist) {
        minCostBT = Integer.MAX_VALUE;
        bestPathBT = "";

        boolean[] visited = new boolean[dist.length];
        visited[0] = true;

        backtrack(0, dist, visited, 1, 0, "UPTM");

        return "Backtracking Route: " + bestPathBT + " | Total Cost: " + minCostBT;
    }

    static void backtrack(int pos, int[][] dist, boolean[] visited,
                          int count, int cost, String path) {

        if (count == dist.length) {
            int totalCost = cost + dist[pos][0];

            if (totalCost < minCostBT) {
                minCostBT = totalCost;
                bestPathBT = path + " -> UPTM";
            }
            return;
        }

        for (int i = 0; i < dist.length; i++) {
            if (!visited[i]) {
                visited[i] = true;

                backtrack(i, dist, visited, count + 1,
                        cost + dist[pos][i],
                        path + " -> " + locations[i]);

                visited[i] = false;
            }
        }
    }

    // divide and conquer 
    static int minCostDAC;
    static String bestPathDAC;

    public static String divideAndConquerMCOP(int[][] dist) {
        minCostDAC = Integer.MAX_VALUE;
        bestPathDAC = "";

        boolean[] visited = new boolean[dist.length];
        visited[0] = true;

        dacHelper(0, visited, dist, 1, 0, "UPTM");

        return "Divide & Conquer Route: " + bestPathDAC + " | Total Cost: " + minCostDAC;
    }

    static void dacHelper(int pos, boolean[] visited, int[][] dist,
                          int count, int cost, String path) {

        if (count == dist.length) {
            int totalCost = cost + dist[pos][0];

            if (totalCost < minCostDAC) {
                minCostDAC = totalCost;
                bestPathDAC = path + " -> UPTM";
            }
            return;
        }

        for (int i = 0; i < dist.length; i++) {
            if (!visited[i]) {
                visited[i] = true;

                dacHelper(i, visited, dist, count + 1,
                        cost + dist[pos][i],
                        path + " -> " + locations[i]);

                visited[i] = false;
            }
        }
    }

    // insertion sort
    public static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int key = arr[i];
            int j = i - 1;

            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }

            arr[j + 1] = key;
        }
    }

    // ================= BINARY SEARCH =================
    public static int binarySearch(int[] arr, int target) {
        int left = 0, right = arr.length - 1;

        while (left <= right) {
            int mid = (left + right) / 2;

            if (arr[mid] == target) return mid;
            else if (arr[mid] < target) left = mid + 1;
            else right = mid - 1;
        }
        return -1;
    }

    //min heap
    static class MinHeap {
        int[] heap;
        int size;

        MinHeap(int capacity) {
            heap = new int[capacity];
            size = 0;
        }

        void insert(int val) {
            heap[size] = val;
            int i = size++;
            while (i > 0 && heap[(i - 1) / 2] > heap[i]) {
                int temp = heap[i];
                heap[i] = heap[(i - 1) / 2];
                heap[(i - 1) / 2] = temp;
                i = (i - 1) / 2;
            }
        }

        int extractMin() {
            if (size == 0) return -1;

            int root = heap[0];
            heap[0] = heap[--size];
            heapify(0);
            return root;
        }

        void heapify(int i) {
            int smallest = i, left = 2 * i + 1, right = 2 * i + 2;

            if (left < size && heap[left] < heap[smallest]) smallest = left;
            if (right < size && heap[right] < heap[smallest]) smallest = right;

            if (smallest != i) {
                int temp = heap[i];
                heap[i] = heap[smallest];
                heap[smallest] = temp;
                heapify(smallest);
            }
        }
    }

    // splay tree
    static class SplayTree {

        class Node {
            int key;
            Node left, right;
            Node(int key) { this.key = key; }
        }

        Node root;

        Node rightRotate(Node x) {
            Node y = x.left;
            x.left = y.right;
            y.right = x;
            return y;
        }

        Node leftRotate(Node x) {
            Node y = x.right;
            x.right = y.left;
            y.left = x;
            return y;
        }

        Node splay(Node root, int key) {
            if (root == null || root.key == key) return root;

            if (key < root.key) {
                if (root.left == null) return root;

                if (key < root.left.key) {
                    root.left.left = splay(root.left.left, key);
                    root = rightRotate(root);
                } else if (key > root.left.key) {
                    root.left.right = splay(root.left.right, key);
                    if (root.left.right != null)
                        root.left = leftRotate(root.left);
                }

                return (root.left == null) ? root : rightRotate(root);
            } else {
                if (root.right == null) return root;

                if (key > root.right.key) {
                    root.right.right = splay(root.right.right, key);
                    root = leftRotate(root);
                } else if (key < root.right.key) {
                    root.right.left = splay(root.right.left, key);
                    if (root.right.left != null)
                        root.right = rightRotate(root.right);
                }

                return (root.right == null) ? root : leftRotate(root);
            }
        }

        Node insert(Node root, int key) {
            if (root == null) return new Node(key);

            root = splay(root, key);

            if (root.key == key) return root;

            Node newNode = new Node(key);

            if (key < root.key) {
                newNode.right = root;
                newNode.left = root.left;
                root.left = null;
            } else {
                newNode.left = root;
                newNode.right = root.right;
                root.right = null;
            }

            return newNode;
        }

        boolean search(int key) {
            root = splay(root, key);
            return (root != null && root.key == key);
        }
    }

    //main
    public static void main(String[] args) {

        System.out.println("Sample output:\n");

        System.out.println(greedyMCOP(costMatrix));
        System.out.println(dynamicProgrammingMCOP(costMatrix));
        System.out.println(backtrackingMCOP(costMatrix));
        System.out.println(divideAndConquerMCOP(costMatrix));

        int[] arr = {8, 3, 5, 1, 9, 2};
        insertionSort(arr);
        System.out.println("Sorted Array: " + Arrays.toString(arr));

        int index = binarySearch(arr, 5);
        System.out.println("Binary Search (5 found at index): " + index);

        // MIN HEAP
        MinHeap heap = new MinHeap(10);
        heap.insert(10);
        heap.insert(3);
        heap.insert(15);
        heap.insert(20);

        System.out.println("Min-Heap Extract Min: " + heap.extractMin());

        // SPLAY TREE
        SplayTree tree = new SplayTree();
        tree.root = tree.insert(tree.root, 10);
        tree.root = tree.insert(tree.root, 20);
        tree.root = tree.insert(tree.root, 5);
        tree.root = tree.insert(tree.root, 15);

        boolean found = tree.search(10);
        System.out.println("Splay Tree Search (10 found): " + found);
    }
}
