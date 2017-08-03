---
title: 最短路径之迪杰斯特拉算法
date: 2017-07-28 23:36:29
tags: 算法
categories: 算法
---

```c++
#include <iostream>
#include <fstream>
#include <vector>

using std::cin;
using std::cout;
using std::vector;

//maximum number of nodes
const int maxnum = 100;
//maxWeight represents infinite large
const int maxWeight = 99999;

///
///Calculate the shortest path based on dijstra algorithm.
///
///@param v source node index
///@param n number of nodes
///@param dist to save the distance between the ith node and the source node 
///@param record the ith node's previous node.
///@param arcs line matrix
void dijstra(int v, int n, int *dist, int *prev, int arcs[maxnum][maxnum]) {
    //bool array marks which node has been put to S.
    //S is an array that contains nodes which makes up of the shortest path.
    bool included[n];

    for(int i = 0; i < n; i++) {
        included[i] = false;
        dist[i] = arcs[v][i];

        //if the distance is not maxWeight, then it means there 
        //is a line between the v and the ith node. so set the ith
        //node's previous to v.
        if(dist[i] < maxWeight) {
            prev[i] = v;
        } else {
            prev[i] = -1;
        }
    }

    //start from the source node v, so put it to S by default.
    included[v] = true;
    //self to self is 0.
    dist[v] = 0;

    for(int i = 1; i < n; i++) {
        int min = maxWeight;

        int w;
        //find min distance between the source node and the ith
        //node. the ith node must not be in S.
        for(int j = 0; j < n; j++) {
            if(!included[j] && dist[j] < min) {
                min = dist[j];
                w = j;
            }
        }

        //put the w node into the S.
        included[w] = true;

        for(int j = 0; j < n; j++) {
            if(!included[j] && min + arcs[w][j] < dist[j]) {
                dist[j] = min + arcs[w][j];
                prev[j] = w;
            }
        }
    }
}

int main() {
    std::ifstream ifin;
    //number of nodes
    int n;
    //number of lines
    int lines;
    int arcs[maxnum][maxnum];

    ifin.open("sample.txt");
    if(ifin.is_open()) {
        ifin >> n;
        ifin >> lines;

        for(int i = 0; i < lines; i++) {
            int row, column, weight;
            ifin >> row >> column >> weight;

            arcs[row][column] = weight;
        }
    } else {
        cout << "could not open sample.txt " << std::endl;
        return 0;
    }

    ifin.close();

    cout << n << " nodes " << std::endl;
    cout << lines << " lines " << std::endl;

    //to save the distance between the ith node and the source node 
    int dist[n];
    //to record the ith node's previous node.
    int prev[n];

    //init arcs
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            if(arcs[i][j] == 0) {
                arcs[i][j] = maxWeight;
            }
            cout << arcs[i][j] << "\t";
        }

        cout << std::endl;
    }

    const int v = 0;
    dijstra(v, n, dist, prev, arcs);

    cout << "distance between "<< v <<" and last point is :" << dist[n-1] << std::endl;

    cout << "shortest path is [";
    int k = n - 1;
    vector<int> path(1,k);
    while(prev[k] >= 0) {
        k = prev[k];
        path.push_back(k);
    }
    
    for(vector<int>::reverse_iterator iter = path.rbegin(); iter != path.rend(); ++iter) {
        cout << *iter << " ";
    }

    cout << "]" << std::endl;
    
    return 0;
}
```