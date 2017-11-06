#include<stdio.h>
#include<stdlib.h>
typedef struct Node {
	int i, j, e;
	struct Node* down, *right;
}Node,*Olink;
//定义矩阵元素类型及其指针

typedef struct CrossMatrix {
	Olink * mr, *mc;//行，列的头结点
	int mu, nu, tu;//矩阵的行，列，总元素个数
}CrossMatrix;
void InitCM(CrossMatrix*);//初始化十字链表，先读取矩阵行列数，再按行，列，元素值读取元素
int Attachrow(Olink p,CrossMatrix* a, int i, int j);//行插入函数
int Attachcol(Olink p,CrossMatrix* a, int i, int j);//列插入函数
void Sum(CrossMatrix*,  CrossMatrix*);//将参数位置上第二个矩阵加到第一个上
void Print(const CrossMatrix*);
int main() {
	CrossMatrix a, b;
	InitCM(&a);
	InitCM(&b);
	Sum(&a, &b);//把b矩阵加到a上
	Print(&a);
	return 0;
}

void InitCM(CrossMatrix* a) {
	int m = 0, n = 0;
	printf("请输入矩阵行数，列数\n");
	scanf("%d %d", &m, &n);
	a->mu = m;
	a->nu = n;
	a->mr = (Olink*)malloc(sizeof(Olink)*(m + 1));//行头指针
	a->mc = (Olink*)malloc(sizeof(Olink)*(n + 1));//列头指针
	for (int i = 0;i < m + 1;i++) {
		(a->mc)[i] = NULL;
	}         
	for (int i = 0;i < n + 1;i++) {
		a->mr[i] = NULL;
	}//行，列头指针清零//头指针的第0位用不到了
	int i = 0, j = 0, e = 0;//保存矩阵元素行数，列数，值
	printf("请依次输入矩阵元素的行数，列数，元素值,（某组输入中的行数若为零值\
则终止输入）:\n");
	scanf("%d %d %d", &i, &j, &e);
	while (i != 0) {
		Olink p = (Olink)malloc(sizeof(Node));
		p->i = i;
		p->j = j;
		p->e = e;//初始化结点，接下来先行插入后列插入
		int ret = 0;
		ret = Attachrow(p, a, i, j);//把p加入矩阵a的行中
		if (ret == 1) {//说明出现了元素重复，重新读一次
			scanf("%d %d %d", &i, &j, &e);
			continue;
		}
		ret = Attachcol(p, a, i, j);//把p加入矩阵a的列中，行已经判断过了所以不会出现元素重复了
		scanf("%d %d %d", &i, &j, &e);
	}
	
}

int Attachrow(Olink p, CrossMatrix* a, int i, int j) {
	if (a->mr[i] == NULL || a->mr[i]->j > j) {//可以直接行插入的情况
		p->right = a->mr[i];
		a->mr[i] = p;
	}
	else if (a->mr[i]->j == j) {//重复输入元素的情况
		printf("重复输入了元素！请确认后再输一次：\n");
		return 1;
	}
	else {//查找应插入的位置注意此处与直接插的区别是辅助指针要指向目标结点的前驱
		Olink tmp = NULL;
		for (tmp = a->mr[i];tmp->right &&tmp->right->j < j;tmp = tmp->right);
		if (tmp->j == j) {
			printf("重复输入了元素！请确认后再输一次\n");
			return 1;
		}
		else {
			p->right = tmp->right;
			tmp->right = p;
		}
	}//行插入结束
	return 0;
}
int Attachcol(Olink p, CrossMatrix* a, int j, int i) {//此处参数表对读进的行列做了一个交换，复制上面的代码就少了一些修改工作。。

	if (a->mc[i] == NULL || a->mc[i]->j > j) {//可以直接列插入的情况
		p->down= a->mc[i];
		a->mc[i] = p;
	}
	else if (a->mc[i]->j == j) {//重复输入元素的情况
		printf("重复输入了元素！请确认后再输一次：\n");
		return 1;
	}
	else {//查找应插入的位置注意此处与直接插的区别是辅助指针要指向目标结点的前驱
		Olink tmp = NULL;
		for (tmp = a->mc[i];tmp->down &&tmp->down->j < j;tmp = tmp->down);
		if (tmp->j == j) {
			printf("重复输入了元素！请确认后再输一次\n");
			return 1;
		}
		else {
			p->down = tmp->down;
			tmp->down = p;
		}
	}//列插入结束
	return 0;
}
void Sum(CrossMatrix* a, CrossMatrix* b) {//因为有链表的插入操作，所以必须注意前驱结点问题
	if (a->mu != b->mu || a->nu != b->nu) {
		printf("两矩阵行，列数不同，不合法的加法");
		return;
	}
	Olink ap = a->mr[1], bp = b->mr[1];//ap,bp分别是指向a,b矩阵的第一行头的指针
	Olink pre = NULL;//ap指针的前置指针
	Olink* up=(Olink*)malloc(((a->nu) + 1) * sizeof(Olink));//用于做ap顶置指针的预备指针列（因为ap上方的指针不好跟ap同时移动所以做一个指针列以随时使用）
	for (int i = 0;i < a->nu;i++) {
		up[i] = a->mc[i];
	}
	//开始对a，b中元素进行循环遍历做加法
	for (int i = 1;i < (a->mu) + 1;i++) {//对行做遍历
		while (bp != NULL) {//对b的列做遍历
			Olink p = (Olink)malloc(sizeof(Node));
			*p = *bp;//把bp所指内容复制到p所指结点中
			/* 情况1  */if (ap == NULL || ap->j > p->j) {//*p可以直接插入操作的情况
				if (pre == NULL) {//此处是分类讨论的技巧，省代码了，但其实可读性变差
					a->mr[i]->right = p;
				}
				else {
					pre->right = p;
				}
				p->right = ap;
				pre = p;//行插入完成，接下来进行列插入
				if (a->mc[p->j] == NULL || a->mc[p->j]->i > p->i) {//可以直接插的情况
					p->down = a->mc[p->j];
					a->mc[p->j] = p;
				}
				else {
					for (;up[i]->down->i < i;up[i] = up[i]->down);//找到该插入的上方结点位置
					p->down = up[i]->down;
					up[i]->down = p;
				}//列插入完成，对bp移位
				bp = bp->right;
				*p = *bp;
			}

			/*  情况2*/	else if (ap->j < p->j) {//ap需要继续向前移动
				pre = ap;
				ap = ap->right;
			}
			/*情况3  */	else if (ap->j == p->j) {//需要进行元素加法的情况
				if (ap->e + p->e != 0) {//如果相加不为零，则可直接加
					ap->e = ap->e + p->e;//加法完成，对bp移位
					bp = bp->right;
					*p = *bp;
				}
				else {//相加为零，进行删除操作
					if (pre == NULL) {
						a->mr[ap->i] = a->mr[ap->i]->right;
					}
					else {
						pre->right = ap->right;
					}
					Olink t = p;//为列断链加释放操作做铺垫
					p = ap;
					ap = ap->right;
					//进行列断链
					if (a->mc[p->j]->i  == i) {//假如可以直接断链
						a->mc[p->j] = a->mc[p->j]->down;
					}
					else {
						for (;up[i]->down->i < i;up[i] = up[i]->down);//找到该位置上方结点的位置
						up[i]->down = up[i]->down->down;
					}
					free(t);
					free(p);
					//删除完成，对bp进行移位
					bp = bp->right;
					*p = *bp;

				}
			}
		}
	}
	return ;
}
void Print(const CrossMatrix* a) {
	Olink rp = a->mr[1];
	for (int i = 1;i < (a->mu) + 1;i++) {
		while (rp != NULL) {
			printf("(%d行,%d列,值：%d)\t", rp->i, rp->j, rp->e);
		}
		printf("\n");
	}return;
}
