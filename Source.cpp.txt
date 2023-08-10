//Hamza Tariq 21I-0396
//Ibrahim Bin Naeem 21I-0512 
//Ali Musharaf 21I-1384

#include <conio.h>
#include <stdio.h>
#include<iostream>
#include <string>
#include <queue>
#include<Windows.h>        //Library for colors and sound
#include <playsoundapi.h>  //library for sound (link the library in the project first)
#include <mmsystem.h>


using namespace std;

const int INF = 1e9;

const int map_size = 20;

void printmap(string gp[map_size]);

class Node {
public:
    int id;
    int reward_score;
    int count;
    Node* left;
    Node* right;
    int height;

    Node(int id, int reward_score) : id(id), reward_score(reward_score), count(1), left(nullptr), right(nullptr), height(1) {}
};

class AVLTree {
public:
    Node* root;

    int height(Node* N) {
        if (N == nullptr) {
            return 0;
        }
        return N->height;
    }

    int balanceFactor(Node* N) {
        if (N == nullptr) {
            return 0;
        }
        return height(N->left) - height(N->right);
    }

    Node* rightRotate(Node* y) {
        Node* x = y->left;
        Node* T2 = x->right;

        x->right = y;
        y->left = T2;

        y->height = max(height(y->left), height(y->right)) + 1;
        x->height = max(height(x->left), height(x->right)) + 1;

        return x;
    }

    Node* leftRotate(Node* x) {
        Node* y = x->right;
        Node* T2 = y->left;

        y->left = x;
        x->right = T2;

        x->height = max(height(x->left), height(x->right)) + 1;
        y->height = max(height(y->left), height(y->right)) + 1;

        return y;
    }

    Node* insertHelper(Node* node, int id, int reward_score) {
        if (node == nullptr) {
            return new Node(id, reward_score);
        }

        if (id == node->id) {
            node->count++;
            return node;
        }

        if (id < node->id) {
            node->left = insertHelper(node->left, id, reward_score);
        }
        else {
            node->right = insertHelper(node->right, id, reward_score);
        }

        node->height = max(height(node->left), height(node->right)) + 1;
        int balance = balanceFactor(node);

        if (balance > 1) {
            if (id < node->left->id) {
                return rightRotate(node);
            }
            else {
                node->left = leftRotate(node->left);
                return rightRotate(node);
            }
        }

        if (balance < -1) {
            if (id > node->right->id) {
                return leftRotate(node);
            }
            else {
                node->right = rightRotate(node->right);
                return leftRotate(node);
            }
        }

        return node;
    }

    Node* minValueNode(Node* node) {
        Node* current = node;

        while (current->left != nullptr) {
            current = current->left;
        }

        return current;
    }
   

    Node* removeClosestUtil(Node* node, int value, Node*& removed_node) {
        if (node == nullptr) {
            return nullptr;
        }

        int diff = abs(node->reward_score - value);

        if (removed_node == nullptr || abs(removed_node->reward_score - value) > diff) {
            removed_node = node;
        }

        if (node->reward_score < value) {
            if (node->right != nullptr) {
                removeClosestUtil(node->right, value, removed_node);
            }
        }
        else if (node->reward_score > value) {
            if (node->left != nullptr) {
                removeClosestUtil(node->left, value, removed_node);
            }
        }

        return node;
    }

   
    int removeClosest(int value) {
        Node* removed_node = nullptr;
        removeClosestUtil(root, value, removed_node);

        if (removed_node != nullptr) {
            cout << "Closest node id: " << removed_node->id << " score: " << removed_node->reward_score << endl;
            int removed_reward_score = removed_node->reward_score;
            root = removeNode(root, removed_node->id);
            return removed_reward_score;
        }
        cout << "No node removed" << endl;
        return -1;
    }



    Node* removeHelper(Node* root, int id) {
        if (root == nullptr) {
            return root;
        }

        if (id < root->id) {
            root->left = removeHelper(root->left, id);
        }
        else if (id > root->id) {
            root->right = removeHelper(root->right, id);
        }
        else {
            if ((root->left == nullptr) || (root->right == nullptr)) {
                Node* temp = root->left ? root->left : root->right;

                if (temp == nullptr)
                {
                    temp = root;
                    root = nullptr;
                }
                else {
                    *root = *temp;
                }

                delete temp;
            }
            else {
                Node* temp = minValueNode(root->right);
                root->id = temp->id;
                root->reward_score = temp->reward_score;
                root->count = temp->count;
                root->right = removeHelper(root->right, temp->id);
            }
        }


        if (root == nullptr) {
            return root;
        }

        root->height = max(height(root->left), height(root->right)) + 1;
        int balance = balanceFactor(root);

        if (balance > 1) {
            if (balanceFactor(root->left) >= 0) {
                return rightRotate(root);
            }
            else {
                root->left = leftRotate(root->left);
                return rightRotate(root);
            }
        }

        if (balance < -1) {
            if (balanceFactor(root->right) <= 0) {
                return leftRotate(root);
            }
            else {
                root->right = rightRotate(root->right);
                return leftRotate(root);
            }
        }

        return root;
    }

    Node* findClosestUtil(Node* node, int target, Node*& closest) {
        if (node == nullptr) {
            return closest;
        }

        // If the target value is equal to the current node's reward_score, return the node
        if (node->reward_score == target) {
            return node;
        }

        // Update the closest node if the current node's reward_score is closer to the target value
        if (closest == nullptr || abs(node->reward_score - target) < abs(closest->reward_score - target)) {
            closest = node;
        }

        // Traverse the left or right subtree based on the target value
        if (target < node->reward_score) {
            return findClosestUtil(node->left, target, closest);
        }
        else {
            return findClosestUtil(node->right, target, closest);
        }
    }

    Node* findClosest(int target) {
        Node* closest = nullptr;
        return findClosestUtil(root, target, closest);
    }

    void preOrderHelper(Node* root) {
        if (root != nullptr) {
            cout << "(" << root->id << ", " << root->reward_score << ", " << root->count << ") ";
            preOrderHelper(root->left);
            preOrderHelper(root->right);
        }
    }

public:
    AVLTree() : root(nullptr) {}

    void insert(int id, int reward_score) {
        // cout << "Inserting id: " << id << " reward_score: " << reward_score << endl;
        root = insertHelper(root, id, reward_score);
    }

    void remove(int id) {
        root = removeHelper(root, id);
    }

    void preOrder() {
        preOrderHelper(root);
        cout << endl;
    }
    Node* removeNode(Node* node, int id) {
        if (node == nullptr) {
            return nullptr;
        }

        if (id < node->id) {
            node->left = removeNode(node->left, id);
        }
        else if (id > node->id) {
            node->right = removeNode(node->right, id);
        }
        else {
            Node* temp;
            if (node->left == nullptr) {
                temp = node->right;
                delete node;
                return temp;
            }
            else if (node->right == nullptr) {
                temp = node->left;
                delete node;
                return temp;
            }

            temp = minValueNode(node->right);
            node->reward_score = temp->reward_score;
            node->id = temp->id;
            node->right = removeNode(node->right, temp->id);
        }

        if (node == nullptr) {
            return node;
        }

        node->height = 1 + max(height(node->left), height(node->right));
        int balance = balanceFactor(node);

        if (balance > 1 && balanceFactor(node->left) >= 0) {
            return rightRotate(node);
        }

        if (balance > 1 && balanceFactor(node->left) < 0) {
            node->left = leftRotate(node->left);
            return rightRotate(node);
        }

        if (balance < -1 && balanceFactor(node->right) <= 0) {
            return leftRotate(node);
        }

        if (balance < -1 && balanceFactor(node->right) > 0) {
            node->right = rightRotate(node->right);
            return leftRotate(node);
        }

        return node;
    }

};




int getIndex(int i, int j) {
    return i * map_size + j;
}

bool isValid(int i, int j) {
    return i >= 0 && i < map_size&& j >= 0 && j < map_size;
}
int getRewardOrPenalty(char c) {
    switch (c) {
    case 'J': return 50; // Jewel
    case 'P': return 70; // Potion
    case 'W': return 30; // Weapon
    case '@': return -30; // Werewolf
    case '$': return -70; // Goblin
    case '&': return -50; // Dragon
    default:

        return 0;
    }
}

void display_scores(const int path[], int path_length, const string game_map[]) {
    int score = 0;
    for (int i = 0; i < path_length; i++) {
        int row = path[i] / map_size;
        int col = path[i] % map_size;
        char cell = game_map[row][col];
        int reward_or_penalty = getRewardOrPenalty(cell);
        score += reward_or_penalty;
        cout << "Step " << i + 1 << ": (" << row << ", " << col << ") " << "Score: " << score << endl;
    }
    cout << "Final score: " << score << endl;
}

void create_adjacency_matrix(const string game_map[], int adj_matrix[][map_size * map_size]) {
    for (int i = 0; i < map_size; i++) {
        for (int j = 0; j < map_size; j++) {
            for (int x = 0; x < map_size; x++) {
                for (int y = 0; y < map_size; y++) {
                    if (i == x && j == y) {
                        adj_matrix[getIndex(i, j)][getIndex(x, y)] = 0;
                    }
                    else {
                        adj_matrix[getIndex(i, j)][getIndex(x, y)] = INF;
                    }
                }
            }
        }
    }

    const int dx[] = { 1, 0, -1, 0 };
    const int dy[] = { 0, 1, 0, -1 };

    for (int i = 0; i < map_size; i++) {
        for (int j = 0; j < map_size; j++) {
            for (int k = 0; k < 4; k++) {
                int ni = i + dx[k];
                int nj = j + dy[k];

                if (isValid(ni, nj)) {
                    int weight = 1;
                    if (game_map[ni][nj] == '#') {
                        weight = 100;
                    }
                    //else if (game_map[ni][nj] == '%') {
                    //    weight = INF; // Set the weight to INF for death points to make them unreachable
                    //}
                    adj_matrix[getIndex(i, j)][getIndex(ni, nj)] = weight;
                }
            }
        }
    }
}




bool is_death_point(int node, const string game_map[]) {
    int row = node / map_size;
    int col = node % map_size;
    return game_map[row][col] == '%';
}

void floyd_warshall(int adj_matrix[][map_size * map_size], int shortest_paths[][map_size * map_size], const string game_map[], int path[], int& path_length, int start_row, int start_col)
{
    // Initialize predecessor matrix
    int** predecessor = new int* [map_size * map_size];
    for (int i = 0; i < map_size * map_size; i++) {
        predecessor[i] = new int[map_size * map_size];
    }

    // Initialize shortest paths matrix and predecessor matrix
    for (int i = 0; i < map_size * map_size; i++) {
        for (int j = 0; j < map_size * map_size; j++) {
            shortest_paths[i][j] = adj_matrix[i][j];
            predecessor[i][j] = (i == j) ? -1 : i;
        }
    }

    // Compute shortest paths
    for (int k = 0; k < map_size * map_size; k++) {
        if (is_death_point(k, game_map)) {
            continue;
        }
        for (int i = 0; i < map_size * map_size; i++) {
            for (int j = 0; j < map_size * map_size; j++) {
                if (shortest_paths[i][k] != INF && shortest_paths[k][j] != INF) {
                    int new_dist = shortest_paths[i][k] + shortest_paths[k][j];

                    if (new_dist < shortest_paths[i][j]) {
                        shortest_paths[i][j] = new_dist;
                        predecessor[i][j] = predecessor[k][j];
                    }
                }
            }
        }
    }

    // Find the position of the crystal
    int crystal_i = -1;
    int crystal_j = -1;
    for (int i = 0; i < map_size; i++) {
        for (int j = 0; j < map_size; j++) {
            if (game_map[i][j] == '*') {
                crystal_i = i;
                crystal_j = j;
                break;
            }
        }
        if (crystal_i != -1 && crystal_j != -1) {
            break;
        }
    }

    // Retrieve the path
    int src = getIndex(start_row, start_col);  // Use start_row and start_col instead of (0, 0)
    int dest = getIndex(crystal_i, crystal_j);

    if (shortest_paths[src][dest] == INF) {
        path_length = 0;
    }
    else {
        path_length = 0;
        int u = dest;
        while (u != -1) {
            path[path_length++] = u;
            u = predecessor[src][u];
        }
        // Reverse the path array since we have retrieved the path in reverse order
        for (int i = 0; i < path_length / 2; i++) {
            swap(path[i], path[path_length - i - 1]);
        }
    }

    // Clean up predecessor matrix
    for (int i = 0; i < map_size * map_size; i++) {
        delete[] predecessor[i];
    }
    delete[] predecessor;
}




void prim_mst(int adj_matrix[][map_size * map_size], int mst[][2], const string game_map[]) {
    bool visited[map_size * map_size] = { false };
    int key[map_size * map_size];
    int parent[map_size * map_size];

    // Initialize key and parent arrays
    for (int i = 0; i < map_size * map_size; i++) {
        key[i] = INF;
        parent[i] = -1;
    }

    key[0] = 0;

    // Main loop
    for (int count = 0; count < map_size * map_size - 1; count++) {
        // Find vertex with minimum key value among unvisited vertices
        int min_key = INF;
        int u = -1;

        for (int v = 0; v < map_size * map_size; v++) {
            if (!visited[v] && key[v] < min_key) {
                min_key = key[v];
                u = v;
            }
        }

        visited[u] = true;

        // Update key and parent values of adjacent vertices
        for (int v = 0; v < map_size * map_size; v++) {
            int u_row = u / map_size;
            int u_col = u % map_size;
            int v_row = v / map_size;
            int v_col = v % map_size;

            if (v != u && v >= 0 && v < map_size * map_size && !visited[v] && adj_matrix[u][v] != 0) {
                int reward_or_penalty = getRewardOrPenalty(game_map[v_row][v_col]);

                if (adj_matrix[u][v] + reward_or_penalty < key[v]) {
                    parent[v] = u;
                    key[v] = adj_matrix[u][v] + reward_or_penalty;
                }
            }
        }
    }

    // Construct MST using parent array
    for (int i = 1; i < map_size * map_size; i++) {
        mst[i - 1][0] = parent[i];
        mst[i - 1][1] = i;
    }
}





int calculate_shortest_path_length(int adj_matrix[][map_size * map_size], const int path[], int path_length) {
    int shortest_path_length = 0;

    for (int i = 0; i < path_length - 1; i++) {
        int u = path[i];
        int v = path[i + 1];
        shortest_path_length += adj_matrix[u][v];
    }

    return shortest_path_length;
}





void print_visited_map(const string game_map[], const bool visited[]) {
    string visual_map[map_size];
    for (int i = 0; i < map_size; i++) {
        visual_map[i] = game_map[i];
    }

    // Mark the visited nodes on the visual_map
    for (int i = 0; i < map_size * map_size; i++) {
        if (visited[i]) {
            int row = i / map_size;
            int col = i % map_size;
            visual_map[row][col] = '+';
        }
    }

    // Print the visual_map
    printmap(visual_map);
}




void dijkstra(int adj_matrix[][map_size * map_size], int src, int dist[], const string game_map[], int path[], int& path_length) {
    const wchar_t* d = L"C:\\Users\\dell\\OneDrive\\Desktop\\projectData\\Epic-Emotional-Orchestral-Music-The-Quest-_Royalty-Free_-_1_.wav";
    PlaySound(d, NULL, SND_ASYNC | 1);

    bool visited[map_size * map_size] = { false };
    int parent[map_size * map_size];

    for (int i = 0; i < map_size * map_size; i++) {
        dist[i] = INF;
        parent[i] = -1;
    }

    dist[src] = 0;
    int crystal_index = -1;

    for (int i = 0; i < map_size; i++) {
        for (int j = 0; j < map_size; j++) {
            if (game_map[i][j] == '*') {
                crystal_index = getIndex(i, j);
                break;
            }
        }
        if (crystal_index != -1) {
            break;
        }
    }

    bool found_crystal = false;

    while (!found_crystal) {
        int u = -1;
        int min_dist = INF;

        for (int i = 0; i < map_size * map_size; i++) {
            if (!visited[i] && dist[i] < min_dist) {
                min_dist = dist[i];
                u = i;
            }
        }

        if (u == -1) {
            break;
        }

        visited[u] = true;

        if (u == crystal_index) {
            found_crystal = true;
        }

        int row = u / map_size;
        int col = u % map_size;

        if (game_map[row][col] == '%') {
            continue;
        }

        print_visited_map(game_map, visited);
        Sleep(200);
        system("CLS");

        for (int v = 0; v < map_size * map_size; v++) {
            if (adj_matrix[u][v] != INF && !visited[v] && dist[u] + adj_matrix[u][v] < dist[v]) {
                dist[v] = dist[u] + adj_matrix[u][v];
                parent[v] = u;
            }
        }
    }

    if (found_crystal) {
        int destination = crystal_index;
        path_length = 0;

        for (int current = destination; current != -1; current = parent[current]) {
            path[path_length++] = current;
        }

        for (int i = 0; i < path_length / 2; i++) {
            swap(path[i], path[path_length - 1 - i]);
        }

        print_visited_map(game_map, visited);
    }
    else {
        path_length = 0;
    }
}


void modify_adj_matrix(int adj_matrix[][map_size * map_size], const string game_map[]) {
    for (int i = 0; i < map_size * map_size; i++) {
        for (int j = 0; j < map_size * map_size; j++) {
            if (adj_matrix[i][j] != INF) {
                int rewardOrPenalty = getRewardOrPenalty(game_map[j / map_size][j % map_size]);
                // Assign negative weights for rewards
                adj_matrix[i][j] -= rewardOrPenalty;
            }
        }
    }
}






struct Edge {
    int weight;
    int u;
    int v;
};

int find(int parent[], int i) {
    while (parent[i] != i) {
        i = parent[i];
    }
    return i;
}

bool compare_edges(const Edge& a, const Edge& b) {
    return a.weight < b.weight;
}

void kruskal_mst(int adj_matrix[][map_size * map_size], int mst[][2], const string game_map[]) {
    int num_edges = 0;
    Edge edges[map_size * map_size * 4];

    for (int i = 0; i < map_size * map_size; i++) {
        for (int j = 0; j < map_size * map_size; j++) {
            if (adj_matrix[i][j] != INF && adj_matrix[i][j] != 100) {
                int row_i = i / map_size;
                int col_i = i % map_size;
                int row_j = j / map_size;
                int col_j = j % map_size;

                if (isValid(row_i, col_i) && isValid(row_j, col_j)) {
                    int reward_or_penalty_i = getRewardOrPenalty(game_map[row_i][col_i]);
                    int reward_or_penalty_j = getRewardOrPenalty(game_map[row_j][col_j]);
                    int updated_weight = adj_matrix[i][j] + reward_or_penalty_i + reward_or_penalty_j;

                    if (updated_weight == 0 || (game_map[row_i][col_i] != '#' && game_map[row_j][col_j] != '#')) {
                        edges[num_edges].weight = updated_weight;
                        edges[num_edges].u = i;
                        edges[num_edges].v = j;
                        num_edges++;
                    }
                }
            }
        }
    }

    sort(edges, edges + num_edges, compare_edges);

    int parent[map_size * map_size];
    for (int i = 0; i < map_size * map_size; i++) {
        parent[i] = i;
    }

    int count = 0;
    int i = 0;
    while (count < map_size * map_size - 1 && i < num_edges) {
        int u = edges[i].u;
        int v = edges[i].v;

        int parent_u = find(parent, u);
        int parent_v = find(parent, v);

        if (parent_u != parent_v) {
            mst[count][0] = u;
            mst[count][1] = v;
            count++;

            parent[parent_u] = parent_v;
        }

        i++;
    }
}



void generate_random_map(string game_map[]) {
    srand(time(0));

    for (int i = 0; i < map_size; i++) {
        game_map[i] = string(map_size, 'C');
    }

    int crystal_x = rand() % map_size;
    int crystal_y = rand() % map_size;
    game_map[crystal_x][crystal_y] = '*';

    char rewards[] = { 'J', 'P', 'W' };
    for (int i = 0; i < 25; i++) {
        int reward_x, reward_y;
        do {
            reward_x = rand() % map_size;
            reward_y = rand() % map_size;
        } while (game_map[reward_x][reward_y] != 'C');
        game_map[reward_x][reward_y] = rewards[rand() % 3];
    }

    for (int i = 0; i < 15; i++) {
        int death_x, death_y;
        do {
            death_x = rand() % map_size;
            death_y = rand() % map_size;
        } while (game_map[death_x][death_y] != 'C');
        game_map[death_x][death_y] = '%';
    }

    for (int i = 0; i < 50; i++) {
        int obstacle_x, obstacle_y;
        do {
            obstacle_x = rand() % map_size;
            obstacle_y = rand() % map_size;
        } while (game_map[obstacle_x][obstacle_y] != 'C');
        game_map[obstacle_x][obstacle_y] = '#';
    }

    char monsters[] = { '&', '$', '@' };
    for (int i = 0; i < 20; i++) {
        int monster_x, monster_y;
        do {
            monster_x = rand() % map_size;
            monster_y = rand() % map_size;
        } while (game_map[monster_x][monster_y] != 'C');
        game_map[monster_x][monster_y] = monsters[rand() % 3];
    }
}


void print_path(const string game_map[], const int path[], int path_length) {
    const wchar_t* p = L"C:\\Users\\dell\\OneDrive\\Desktop\\projectData\\Poink sound effect.wav";
    string visual_map[map_size];
    for (int i = 0; i < map_size; i++) {
        visual_map[i] = game_map[i];
    }

    // Mark the starting position
    visual_map[0][0] = 'S';

    // Print the visual_map
    printmap(visual_map);

    cout << "Starting..." << endl;
 
    Sleep(1);

    for (int i = 1; i < path_length - 1; i++) {
        int row = path[i] / map_size;
        int col = path[i] % map_size;
        visual_map[row][col] = '.';
        
        PlaySound(p, NULL, SND_ASYNC | 1);


         // Clear the console
        system("cls");



        printmap(visual_map);

        Sleep(500);
    }
}


void level2(string game_map[]) {
    srand(time(0));

    for (int i = 0; i < map_size; i++) {
        game_map[i] = string(map_size, 'C');
    }

    int crystal_x = rand() % map_size;
    int crystal_y = rand() % map_size;
    game_map[crystal_x][crystal_y] = '*';

    char rewards[] = { 'J', 'P', 'W' };
    for (int i = 0; i < 25; i++) {
        int reward_x, reward_y;
        do {
            reward_x = rand() % map_size;
            reward_y = rand() % map_size;
        } while (game_map[reward_x][reward_y] != 'C');
        game_map[reward_x][reward_y] = rewards[rand() % 3];
    }

    for (int i = 0; i < 20; i++) {
        int death_x, death_y;
        do {
            death_x = rand() % map_size;
            death_y = rand() % map_size;
        } while (game_map[death_x][death_y] != 'C');
        game_map[death_x][death_y] = '%';
    }

    for (int i = 0; i < 100; i++) {
        int obstacle_x, obstacle_y;
        do {
            obstacle_x = rand() % map_size;
            obstacle_y = rand() % map_size;
        } while (game_map[obstacle_x][obstacle_y] != 'C');
        game_map[obstacle_x][obstacle_y] = '#';
    }

    char monsters[] = { '&', '$', '@' };
    for (int i = 0; i < 20; i++) {
        int monster_x, monster_y;
        do {
            monster_x = rand() % map_size;
            monster_y = rand() % map_size;
        } while (game_map[monster_x][monster_y] != 'C');
        game_map[monster_x][monster_y] = monsters[rand() % 3];
    }
}

void level3(string game_map[]) {
    srand(time(0));

    for (int i = 0; i < map_size; i++) {
        game_map[i] = string(map_size, 'C');
    }

    int crystal_x = rand() % map_size;
    int crystal_y = rand() % map_size;
    game_map[crystal_x][crystal_y] = '*';

    char rewards[] = { 'J', 'P', 'W' };
    for (int i = 0; i < 25; i++) {
        int reward_x, reward_y;
        do {
            reward_x = rand() % map_size;
            reward_y = rand() % map_size;
        } while (game_map[reward_x][reward_y] != 'C');
        game_map[reward_x][reward_y] = rewards[rand() % 3];
    }

    for (int i = 0; i < 25; i++) {
        int death_x, death_y;
        do {
            death_x = rand() % map_size;
            death_y = rand() % map_size;
        } while (game_map[death_x][death_y] != 'C');
        game_map[death_x][death_y] = '%';
    }

    for (int i = 0; i < 150; i++) {
        int obstacle_x, obstacle_y;
        do {
            obstacle_x = rand() % map_size;
            obstacle_y = rand() % map_size;
        } while (game_map[obstacle_x][obstacle_y] != 'C');
        game_map[obstacle_x][obstacle_y] = '#';
    }

    char monsters[] = { '&', '$', '@' };
    for (int i = 0; i < 20; i++) {
        int monster_x, monster_y;
        do {
            monster_x = rand() % map_size;
            monster_y = rand() % map_size;
        } while (game_map[monster_x][monster_y] != 'C');
        game_map[monster_x][monster_y] = monsters[rand() % 3];
    }
}

void print_menu()
{
    HANDLE console = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleTextAttribute(console, 4);
    cout << "Please choose an option:\n"
        << "1. Find shortest path using default location (0,0) by Floyd-Warshall\n"
        << "2. Find shortest path using default location (0,0) by Dijkstra\n"
        << "3. Find shortest path using custom location by Floyd-Warshall\n"
        << "4. Find shortest path using custom location by Dijkstra\n"
        << "5. Find minimum spanning tree by Prim's algorithm\n"
        << "6. Find minimum spanning tree by Kruskal's algorithm\n"
        << "0. Exit\n";
    SetConsoleTextAttribute(console, 15);
}




void print_mst(int(*mst)[2], int adj_matrix[][map_size * map_size]) {

    int total_weight = 0;
    cout << "Minimum Spanning Tree Edges:" << endl;
    for (int i = 0; i < map_size * map_size - 1; ++i) {
        int u = mst[i][0];
        int v = mst[i][1];
        int weight = adj_matrix[u][v];
        total_weight += weight;
        cout << "(" << u / map_size << ", " << u % map_size << ") - (" << v / map_size << ", " << v % map_size << ") : Weight = " << weight << endl;
    }
    cout << "Total Weight: " << total_weight << endl;
}

bool is_death_point(const string game_map[], int row, int col) {
    return game_map[row][col] == '%';
}


void parse_input(int adj_matrix[][map_size * map_size], const string game_map[]) {
    bool first_step = 0;
    int choice;
    int shortest_path_length=0;
    cin >> choice;
    bool restart;
    int path[map_size * map_size];
    int path_length;
    int(*shortest_paths)[map_size * map_size] = new int[map_size * map_size][map_size * map_size]();
    int(*mst)[2] = new int[map_size * map_size - 1][2]();
    int dijkstra_paths[map_size * map_size];

    int score = 0;
    AVLTree inventory;
    int r1 = 0;
    int c1 = 0;
    switch (choice) {
    case 1: {
        
        floyd_warshall(adj_matrix, shortest_paths, game_map, path, path_length,r1,c1);
        print_path(game_map, path, path_length);
        cout << "Shortest path using Floyd-Warshall: " << shortest_paths[0][getIndex(map_size - 1, map_size - 1)] << endl;
        cout << "Path length: " << path_length << endl;
        cout << "Path: ";
        for (int i = 0; i < path_length; i++) {
            cout << path[i] << " ";
        }
        cout << endl;

        // Display scores
        restart = false;
        first_step = true; // Add a flag for the first step

        for (int i = 0; i < path_length; i++) {
            int row = path[i] / map_size;
            int col = path[i] % map_size;

            if (is_death_point(game_map, row, col)) {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") You have encountered a death point! Restarting the game...\n";
                restart = true;
                break;
            }

            int reward_or_penalty = getRewardOrPenalty(game_map[row][col]);

            if (reward_or_penalty > 0) {
                //sound.play();
                int id;
                if (first_step) { // If it's the first step
                    id = 100; // Set id to 100
                    first_step = false; // Mark the first step as done
                }
                else {
                    id = rand() % 201; // Generate a random id between 0 and 200
                }
                inventory.insert(id, reward_or_penalty);
                score += reward_or_penalty;
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (+" << reward_or_penalty << ")\n";
            }
            else if (reward_or_penalty < 0) {
                int removed_item = inventory.removeClosest(-reward_or_penalty);
                if (removed_item != -1) {
                    score -= removed_item;
                    cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (-" << removed_item << ")\n";
                }
            }
            else {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << "\n";
            }
        }

        if (!restart) {
            cout << "Final score using Dijkstra: " << score << endl;
        }
        else {
            cout << "Restarting game....";
            restart = true;

        }
        break;
    }
    case 2: {
        //modify_adj_matrix(adj_matrix, game_map);
        dijkstra(adj_matrix, getIndex(0, 0), dijkstra_paths, game_map, path, path_length);
        print_path(game_map, path, path_length);
        shortest_path_length = calculate_shortest_path_length(adj_matrix, path, path_length);
        cout << "Shortest path length: " << shortest_path_length << endl;
        cout << "Path length: " << path_length << endl;
        cout << "Path: ";
        for (int i = 0; i < path_length; i++) {
            cout << path[i] << " ";
        }
        cout << endl;

        // Display scores
        restart = false;
        first_step = true; // Add a flag for the first step

        for (int i = 0; i < path_length; i++) {
            int row = path[i] / map_size;
            int col = path[i] % map_size;

            if (is_death_point(game_map, row, col)) {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") You have encountered a death point! Restarting the game...\n";
                restart = true;
                break;
            }

            int reward_or_penalty = getRewardOrPenalty(game_map[row][col]);

            if (reward_or_penalty > 0) {
                //sound.play();
                int id;
                if (first_step) { // If it's the first step
                    id = 100; // Set id to 100
                    first_step = false; // Mark the first step as done
                }
                else {
                    id = rand() % 201; // Generate a random id between 0 and 200
                }
                inventory.insert(id, reward_or_penalty);
                score += reward_or_penalty;
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (+" << reward_or_penalty << ")\n";
            }
            else if (reward_or_penalty < 0) {
                int removed_item = inventory.removeClosest(-reward_or_penalty);
                if (removed_item != -1) {
                    score -= removed_item;
                    cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (-" << removed_item << ")\n";
                }
            }
            else {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << "\n";
            }
        }

        if (!restart) {
            cout << "Final score using Dijkstra: " << score << endl;
        }
        else {
            cout << "Restarting game....";
            restart = true;

        }
        break;
    }
    case 3: {
        int custom_row, custom_col;
        cout << "Enter custom row and column: ";
        cin >> custom_row >> custom_col;

        floyd_warshall(adj_matrix, shortest_paths, game_map, path, path_length,custom_row,custom_col);
        print_path(game_map, path, path_length);
        shortest_path_length = calculate_shortest_path_length(adj_matrix, path, path_length);
        cout << "Shortest path length: " << shortest_path_length << endl;
        cout << "Path: ";
        for (int i = 0; i < path_length; i++) {
            cout << path[i] << " ";
        }
        cout << endl;

        // Display scores
        restart = false;
        first_step = true; // Add a flag for the first step

        for (int i = 0; i < path_length; i++) {
            int row = path[i] / map_size;
            int col = path[i] % map_size;

            if (is_death_point(game_map, row, col)) {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") You have encountered a death point! Restarting the game...\n";
                restart = true;
                break;
            }

            int reward_or_penalty = getRewardOrPenalty(game_map[row][col]);

            if (reward_or_penalty > 0) {
                //sound.play();
                int id;
                if (first_step) { // If it's the first step
                    id = 100; // Set id to 100
                    first_step = false; // Mark the first step as done
                }
                else {
                    id = rand() % 201; // Generate a random id between 0 and 200
                }
                inventory.insert(id, reward_or_penalty);
                score += reward_or_penalty;
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (+" << reward_or_penalty << ")\n";
            }
            else if (reward_or_penalty < 0) {
                int removed_item = inventory.removeClosest(-reward_or_penalty);
                if (removed_item != -1) {
                    score -= removed_item;
                    cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (-" << removed_item << ")\n";
                }
            }
            else {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << "\n";
            }
        }

        if (!restart) {
            cout << "Final score using Dijkstra: " << score << endl;
        }
        else {
            cout << "Restarting game....";
            restart = true;

        }
        break;
    }
    case 4: {
        int custom_row, custom_col;
        cout << "Enter custom row and column: ";
        cin >> custom_row >> custom_col;

        int dijkstra_paths1[map_size * map_size];
        dijkstra(adj_matrix, getIndex(custom_row, custom_col), dijkstra_paths1, game_map, path, path_length);
        print_path(game_map, path, path_length);
        shortest_path_length = calculate_shortest_path_length(adj_matrix, path, path_length);
        cout << "Shortest path length: " << shortest_path_length << endl;

        cout << "Path: ";
        for (int i = 0; i < path_length; i++) {
            cout << path[i] << " ";
        }
        cout << endl;

        // Display scores
        restart = false;
        first_step = true; // Add a flag for the first step

        for (int i = 0; i < path_length; i++) {
            int row = path[i] / map_size;
            int col = path[i] % map_size;

            if (is_death_point(game_map, row, col)) {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") You have encountered a death point! Restarting the game...\n";
                restart = true;
                break;
            }

            int reward_or_penalty = getRewardOrPenalty(game_map[row][col]);

            if (reward_or_penalty > 0) {
                //sound.play();
                int id;
                if (first_step) { // If it's the first step
                    id = 100; // Set id to 100
                    first_step = false; // Mark the first step as done
                }
                else {
                    id = rand() % 201; // Generate a random id between 0 and 200
                }
                inventory.insert(id, reward_or_penalty);
                score += reward_or_penalty;
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (+" << reward_or_penalty << ")\n";
            }
            else if (reward_or_penalty < 0) {
                int removed_item = inventory.removeClosest(-reward_or_penalty);
                if (removed_item != -1) {
                    score -= removed_item;
                    cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << " (-" << removed_item << ")\n";
                }
            }
            else {
                cout << "Step " << i + 1 << ": (" << row << ", " << col << ") Score: " << score << "\n";
            }
        }

        if (!restart) {
            cout << "Final score using Dijkstra: " << score << endl;
        }
        else {
            cout << "Restarting game....";
            restart = true;

        }
        break;
    }
    case 5: {
        prim_mst(adj_matrix, mst, game_map);
        print_mst(mst, adj_matrix);
        break;
    }
    case 6: {
        kruskal_mst(adj_matrix, mst, game_map);
        print_mst(mst, adj_matrix);
        break;
    }
    case 0: {
        cout << "Exiting...\n";
        exit(0);
        break;
    }
    default: {
        cout << "Invalid input. Please try again.\n";
    }
    }
    delete[] shortest_paths;
    delete[] mst;
}


void top()
{
    HANDLE console = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleTextAttribute(console, 3);
    cout << " ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ ____ \n";
    cout << "||T |||H |||E |||  |||Q |||U |||E |||S |||T |||  |||F |||O |||R |||  |||C |||r || \n";
    cout << "||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|||__|| \n";
    cout << "|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\| \n";
    cout << "|    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    | \n";
    cout << "|    |    |    |    |   The Quest for the Crystal Kingdom   |    |    |    |    | \n";
    cout << "|__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  |__  | \n";
    cout << "|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\| \n";
    SetConsoleTextAttribute(console, 15);
}

void printmap(string gp[map_size])
{
    HANDLE console = GetStdHandle(STD_OUTPUT_HANDLE);
    for (int i = 0; i < map_size; i++) {
        for (int j = 0; j < map_size; j++)
        {
            if (gp[i][j] == 'C')
            {

                SetConsoleTextAttribute(console, 15); // set color to green
                cout << gp[i][j];
            }

            else if (gp[i][j] == 'J')
            {

                SetConsoleTextAttribute(console, 4); // set color to green
                cout << gp[i][j];
            }

            else if (gp[i][j] == 'P')
            {

                SetConsoleTextAttribute(console, 1); // set color to green
                cout << gp[i][j];
            }

            else if (gp[i][j] == 'W')
            {

                SetConsoleTextAttribute(console, 8); // set color to green
                cout << gp[i][j];
            }

            else if (gp[i][j] == '%')
            {

                SetConsoleTextAttribute(console, 3); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '#')
            {

                SetConsoleTextAttribute(console, 2); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '&')
            {

                SetConsoleTextAttribute(console, 5); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '$')
            {

                SetConsoleTextAttribute(console, 10); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '@')
            {

                SetConsoleTextAttribute(console, 7); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '*')
            {

                SetConsoleTextAttribute(console, 9); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '.')
            {

                SetConsoleTextAttribute(console, 6); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == 'S')
            {

                SetConsoleTextAttribute(console, 6); // set color to green
                cout << gp[i][j];
            }
            else if (gp[i][j] == '+')
            {

                SetConsoleTextAttribute(console, 8); // set color to green
                cout << gp[i][j];
            }
        }
        cout << endl;
    }
    SetConsoleTextAttribute(console, 15);
}





void THEGAME() {
    system("CLS");
    HANDLE console = GetStdHandle(STD_OUTPUT_HANDLE);
    string game_map[map_size];
    top();
    SetConsoleTextAttribute(console, 4);
    cout << "CHOOSE THE LEVELS YOU WANT TO ENTER" << endl;
    cout << "PRESS 1 FOR LEVEL 1" << endl;
    cout << "PRESS 2 FOR LEVEL 2" << endl;
    cout << "PRESS 3 FOR LEVEL 3" << endl;
    SetConsoleTextAttribute(console, 15);
    int choice;
    cin>>choice;

    if(choice==1)
    {
        generate_random_map(game_map);
    }
    else if (choice == 2)
    {
        level2(game_map);
    }
    else if (choice == 3)
    {
        level3(game_map);
    }
    system("CLS");
;
    
    int adj_matrix[map_size * map_size][map_size * map_size];

    // Display the generated map
    create_adjacency_matrix(game_map, adj_matrix);

    // Print the menu and parse user input
    while (true) {
       
        top();
        const wchar_t* q = L"C:\\Users\\dell\\OneDrive\\Desktop\\projectData\\The Adventure Zone - Crystal Kingdom Song [Complete].wav";
        PlaySound(q, NULL, SND_ASYNC | 1);
        cout << endl << endl;
        cout << "------->THE MAP OF THE FOREST<-------" << endl;
        printmap(game_map);
        

        print_menu();
        parse_input(adj_matrix, game_map);
        char i = _getch();
        system("CLS");
    }
}

//The main runner code
void firstmenu()
{
    HANDLE console = GetStdHandle(STD_OUTPUT_HANDLE);
    system("CLS");
    
    top();
    const wchar_t* q = L"C:\\Users\\dell\\OneDrive\\Desktop\\projectData\\The Adventure Zone - Crystal Kingdom Song [Complete].wav";
    PlaySound(q, NULL, SND_ASYNC | 1);
    int c;
    cout << endl;
    SetConsoleTextAttribute(console, 4);
    cout << "Choose the following: " << endl;
    cout << "Press 1 for Entering the Game:" << endl;
    cout << "press 2 for Showing the Credentials" << endl;
    cout << "press 3 for exit";
    cin >> c;
    SetConsoleTextAttribute(console, 15);
    if (c == 1)
    {
        THEGAME();
    }
    else if (c == 2)
    {
        system("CLS");
        top();
        cout << "Hamza Tariq 21I-0396" << endl;
        cout << "Ali Musharaf Baig 21I-1384" << endl;
        cout << "Ibrahim Bin Naeem 21I-0512" << endl;
        char k = _getch();
        firstmenu();
    }
    else if (c == 3)
    {
        exit(0);
    }
    SetConsoleTextAttribute(console, 15);
}


int main()
{
    firstmenu();

}
