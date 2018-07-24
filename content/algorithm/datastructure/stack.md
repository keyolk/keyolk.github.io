+++
title  = "stack"
toc    = true
weight = 4
+++

## Intro

## Implementation
```
/*
 initialize stack pointer
*/
void init(int *top)
{
    *top = 0;
}
 
/*
 push an element into stack
 precondition: the stack is not full
*/
void push(int *s,int* top, int element)
{
    s[(*top)++] = element;
}
/*
 remove an element from stack
 precondition: stack is not empty
*/
int pop(int *s,int *top)
{
    return s[--(*top)];
}
/*
 return 1 if stack is full, otherwise return 0
*/
int full(int *top,const int size)
{
    return *top == size ? 1 : 0;
}
/*
 return 1 if the stack is empty, otherwise return 0
*/
int empty(int *top)
{
    return *top == 0 ? 1 : 0;
}
 
/*
    display stack content
*/
void display(int *s,int *top)
{
    printf("Stack: ");
    int i;
    for(i = 0; i < *top; i++)
    {
        printf("%d ",s[i]);
    }
    printf("\n");
}
```
