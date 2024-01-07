# database-of-houses
# data about houses in text and binary files will be stored and processed here

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define min(x, y) ( (x) < (y) ? (x) : (y) )
#define LenNum 13

typedef struct home{
    char *string;
    char number[LenNum];
    double square;
}P;

char *readline(){
    char buf[4]={0};
    char *array=NULL;
    int len=0, p=0;
    do{
        p=scanf("%3[^\n]", buf);
        if(p<0){
            if(!array){
                return NULL;
            }
        }
        else if(p>0){
            int len_buf=strlen(buf);
            array=realloc(array, (len+len_buf+1)*sizeof(char));
            memcpy(array+len, buf, len_buf);
            len+=len_buf;
        }
        else{
            scanf("%*c");
        }
    }while(p>0);
    if (len>0){
        array[len]=0;
    }
    else{
        array=calloc(1, sizeof(char));
    }
    return array;
}

void Wind(char *c){
    printf("1. Ввод массива\n2. Сортировка\n3. Вывод массива\n4. Конец\n");
    int p=scanf("%c", c);
    scanf("%*[^\n]");
    scanf("%*c");
    if (p==-1){
        *c='4';
        return ;
    }
    if (p==0 || !(*c>='1' && *c<='4')){
        printf("Не правильный формат ввода, попробуйте ещё!\n");
        Wind(c);
    }
    return ;
}

int correct_number(char *number){
    return (strlen(number)==11 && number[2]==':' && number[5]==':' && number[8]==':' && (number[0]>='0' && number[0]<='9') && (number[1]>='0' && number[1]<='9') && ((number[3]>='a' && number[0]<='z') || (number[3]>='A' && number[3]<='Z')) && ((number[4]>='a' && number[4]<='z') || (number[4]>='A' && number[4]<='Z')) && number[6]=='0' && number[7]=='0' && (number[9]>='0' && number[9]<='9') && (number[10]>='0' && number[10]<='9'));
}

P *Input_terminal(P **arr, int *l){
    if(*l!=0){
        for (int i=0; i<*l; i++){
            free((*arr)[i].string);
        }
        *l=0;
        free(*arr);
        *arr=NULL;
    }
    while (1){
        printf("Введите число элементов: ");
        int p=scanf("%d", l);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==1 && (*l)>0){
            break;
        }
        else{
            if (p==-1){
                return NULL;
            }
            printf("Не правильный формат ввода, попробуйте ещё!\n");
        }
    }
    *arr = (P *)realloc(*arr, *l*sizeof(P));
    if (!arr){
        printf("Не удалось выделить память!\n");
        return NULL;
    }
    int i=0, r;
    P p;
    while (i<(*l)){
        printf("Введите адрес, кадастровый номер и площадь дома с номером %d: ", i+1);
        printf("\nАдрес: ");
        p.string=readline();
        if (p.string==NULL)
            return NULL;
        do{
            printf("Кадастровый номер: ");
            r=scanf("%12s", p.number);
            scanf("%*[^\n]");
            scanf("%*c");
            if(r==-1)
                return NULL;
            r=correct_number(p.number);
        }while(r!=1);
        do{
            printf("Площадь: ");
            r=scanf("%lf", &p.square);
            scanf("%*[^\n]");
            scanf("%*c");
            if (r==-1)
                return NULL;
        }while(!(r==1 && p.square>0.0));
        (*arr)[i]=p;
        i++;
    }
    return *arr;
}
P* Input_txt(P **arr, int *l){
    if (*l!=0){
        for (int i=0; i<*l; i++){
            free((*arr)[i].string);
        }
        free(*arr);
        *arr=NULL;
        *l=0;
    }
    printf("Введите имя файла от куда хотите взять данные: ");
    char *name=readline();
    name=realloc(name, (strlen(name)+5)*sizeof(char));
    strcat(name, ".txt\0");
    FILE *File=fopen(name, "r");
    while (File == NULL) {
        printf("Такого файла нет, попробуйте еще раз!\n");
        free(name);
        name=readline();
        name=realloc(name, (strlen(name)+5)*sizeof(char));
        strcat(name, ".txt");
        File=fopen(name, "r");
    }
    while(File){
        int len_str_p;
        P p;
        if (fscanf(File, "%d", &len_str_p)==EOF){
            break;
        }
        p.string=malloc(sizeof(char));
        p.string[0]=0;
        for (int i=0; i<len_str_p; i++){
            p.string=realloc(p.string, (strlen(p.string)+2)*sizeof(char));
            p.string[i+1]=0;
            fscanf(File, "%c", p.string+i);
        }
        for (int i=0; i<11; i++){
            fscanf(File, "%c", p.number+i);
        }
        p.number[11]=0;
        fscanf(File, "%lf", &(p.square));
        if (correct_number(p.number) && p.square>0.0){
            *l+=1;
            *arr=(P *)realloc(*arr, *l*sizeof(P));
            (*arr)[*l-1]=p;
        }
    }
    fclose(File);
    free(name);
    return (*arr);
}

P* Input_bin(P **arr, int *l){
    if (*l!=0){
        for (int i=0; i<*l; i++){
            free((*arr)[i].string);
        }
        free(*arr);
        *arr=NULL;
        *l=0;
    }
    printf("Введите имя файла от куда хотите взять данные: ");
    char *name=readline();
    name=realloc(name, (strlen(name)+5)*sizeof(char));
    strcat(name, ".bin");
    FILE *File=fopen(name, "rb");
    while (File == NULL) {
        printf("Такого файла нет, попробуйте еще раз!\n");
        free(name);
        name=readline();
        name=realloc(name, (strlen(name)+5)*sizeof(char));
        strcat(name, ".bin");
        File=fopen(name, "rb");
    }
    while(!feof(File)){
        int len_str_p;
        P p;
        if (fread(&len_str_p, sizeof(int), 1, File)==0)
            break;
        p.string=malloc((len_str_p+1)*sizeof(char));
        for (int i=0; i<len_str_p; i++){
            fread(p.string+i, sizeof(char), 1, File);
        }
        p.string[len_str_p]=0;
        for (int i=0; i<11; i++){
            fread(p.number+i, sizeof(char), 1, File);
        }
        p.number[11]=0;
        fread(&(p.square), sizeof(double), 1, File);
        if (correct_number(p.number) && p.square>0.0){
            *l+=1;
            *arr=(P *)realloc(*arr, *l*sizeof(P));
            (*arr)[*l-1]=p;
        }
    }
    fclose(File);
    free(name);
    return (*arr);
}


P* Input_array(P**array, int *len){
    char c;
    int p;
    do{
        printf("1. Ввод массива через терминал\n2. Ввод массива через файл txt\n3. Ввод массива через бинарный файл\n");
        p=scanf("%c", &c);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==-1){
            c=' ';
            return NULL;
        }
        if (p==0 || !(c>='1' && c<='3')){
            p=0;
            printf("Неправильный формат ввода, попробуйте еще раз\n");
        }
    }while(p!=1);
    switch (c){
        case '1':
            Input_terminal(array, len);
            break;
        case '2':
            Input_txt(array, len);
            break;
        case '3':
            Input_bin(array, len);
            break;
    }
    return *array;
}

int COMP(P a, P b, int par, int comp){
    if (comp==1){
        if (par==1)
            return strcmp(a.string, b.string);
        else
            return strcmp(b.string, a.string);
    }
    if (comp==2){
        if (par==1)
            return strcmp(a.number, b.number);
        else
            return strcmp(b.number, a.number);
    }
    else{
        if (par==1)
            return a.square>b.square;
        else
            return a.square<=b.square;
    }
}

P* Bubble_sort(P **arr, int *l, int par, int comp){
    for (int i=0; i<*l; i++){
        for (int j=0; j<*l-1; j++){
            if (COMP((*arr)[j], (*arr)[j+1], par, comp)>0){
                P q=(*arr)[j];
                (*arr)[j]=(*arr)[j+1];
                (*arr)[j+1]=q;
            }
        }
    }
    return (*arr);
}

P* Insertion_sort(P **arr, int *l, int par, int comp){
    P q=(*arr)[0];
    for (int i=1; i<*l; i++){
        for (int j=i; j>0; j--){
            if (COMP((*arr)[j-1], (*arr)[j], par, comp)>0){
                q=(*arr)[j];
                (*arr)[j]=(*arr)[j-1];
                (*arr)[j-1]=q;
            }
        }
    }
    return (*arr);
}

int cmpIst (const void * a, const void * b) {
    return strcmp( (*(P *)a).string, (*(P *)b).string);
}

int cmpDst(const void *a, const void *b){
    return strcmp( (*(P *)b).string, (*(P *)a).string);
}

int cmpInb(const void *a, const void *b){
    return strcmp( (*(P *)a).number, (*(P *)b).number);
}
int cmpDnb(const void *a, const void *b){
    return strcmp( (*(P *)b).number, (*(P *)a).number);
}
int cmpIsq(const void *a_void, const void *b_void){
    P a=*(P *)a_void;
    P b=*(P *)b_void;
    return (int)(a.square-b.square);
}
int cmpDsq(const void *a_void, const void *b_void){
    P a=*(P *)a_void;
    P b=*(P *)b_void;
    return (int)(b.square-a.square);
}
P* Sort_array(P **array, int *len){
    char c, c1, c2;
    int p;
    do{
        printf("1. Bubble sort\n2. Insertion sort\n3. qsort\n");
        p=scanf("%c", &c);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==-1){
            c=' ';
            return NULL;
        }
        if (p==0 || !(c>='1' && c<='3')){
            p=0;
            printf("Неправильный формат ввода, попробуйте еще раз\n");
        }
    }while(p!=1);
    do{
        printf("1. По возрастанию\n2. По убыванию\n");
        p=scanf("%c", &c1);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==-1){
            c1=' ';
            return NULL;
        }
        if (p==0 || !(c1>='1' && c1<='2')){
            p=0;
            printf("Неправильный формат ввода, попробуйте еще раз\n");
        }
    }while(p!=1);
    int par=c1-'0';
    do{
        printf("1. По адресу\n2. По кадастровому номеру\n3. По площади\n");
        p=scanf("%c", &c2);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==-1){
            c2=' ';
            return NULL;
        }
        if (p==0 || !(c2>='1' && c2<='3')){
            p=0;
            printf("Неправильный формат ввода, попробуйте еще раз\n");
        }
    }while(p!=1);
    int comp=c2-'0';
    switch (c){
        case '1':
            Bubble_sort(array, len, par, comp);
            break;
        case '2':
            Insertion_sort(array, len, par, comp);
            break;
        case '3':
            if (comp==1){
                if (par==1)
                    qsort(*array, *len, sizeof(P), cmpIst);
                else
                    qsort(*array, *len, sizeof(P), cmpDst);
            }
            else if (comp==2){
                if (par==1)
                    qsort(*array, *len, sizeof(P), cmpInb);
                else
                    qsort(*array, *len, sizeof(P), cmpDnb);
            }
            else{
                if (par==1)
                    qsort(*array, *len, sizeof(P), cmpIsq);
                else
                    qsort(*array, *len, sizeof(P), cmpDsq);
            }
            break;
    }
    return *array;
}

void Output_terminal(P *arr, int l){
    for (int i=0; i<l; i++){
           printf("%s\n%s\n%lf\n\n", (arr+i)->string, (arr+i)->number, (arr+i)->square);
    }
}

void Output_txt(P *arr, int l){
    printf("Введите имя файла куда хотите записать данные: ");
    char *name=readline();
    name=realloc(name, (strlen(name)+5)*sizeof(char));
    strcat(name, ".txt");
    FILE *pFile=fopen (name,"w");
    while (pFile == NULL) {
        printf("Такого файла нет, попробуйте еще раз!\n");
        free(name);
        name=readline();
        name=realloc(name, (strlen(name)+5)*sizeof(char));
        strcat(name, ".txt");
        pFile=fopen (name,"w");
        
    }
    for (int i=0; i<l; i++){
        fprintf(pFile, "%d%s%s%lf\n", strlen(arr[i].string), arr[i].string, arr[i].number, arr[i].square);
    }
    fclose(pFile);
    free(name);
}
void Output_bin(P *arr, int l){
    printf("Введите имя файла куда хотите записать данные: ");
    char *name=readline();
    name=realloc(name, (strlen(name)+5)*sizeof(char));
    strcat(name, ".bin");
    FILE *pFile=fopen (name,"wb");
    while (pFile == NULL) {
        printf("Такого файла нет, попробуйте еще раз!\n");
        free(name);
        name=readline();
        name=realloc(name, (strlen(name)+5)*sizeof(char));
        strcat(name, ".bin");
        pFile=fopen (name,"wb");
        
    }
    for (int i=0; i<l; i++){
        int lenstr=strlen(arr[i].string);
        fwrite(&lenstr, sizeof(int), 1, pFile);
        fwrite(arr[i].string, sizeof(char), lenstr, pFile);
        fwrite(arr[i].number, sizeof(char), 11, pFile);
        fwrite(&(arr[i].square), sizeof(double), 1, pFile);
    }
    fclose(pFile);
    free(name);
}

P* Output_array(P*array, int len){
    char c;
    int p;
    do{
        printf("1. Вывод массива через терминал\n2. Вывод массива через файл txt\n3. Вывод массива через бинарный файл\n");
        p=scanf("%c", &c);
        scanf("%*[^\n]");
        scanf("%*c");
        if (p==-1){
            return NULL;
        }
        if (p==0 || !(c>='1' && c<='3')){
            p=0;
            printf("Неправильный формат ввода, попробуйте еще раз\n");
        }
    }while(p!=1);
    switch (c){
        case '1':
            Output_terminal(array, len);
            break;
        case '2':
            Output_txt(array, len);
            break;
        case '3':
            Output_bin(array, len);
            break;
    }
    return array;
}

int main() {
    char c;
    int len=0;
    P *arr=NULL;
    Wind(&c);
    while (c!='4'){
        if(c=='1'){
            Input_array(&arr, &len);
            if (arr==NULL && len!=0){
                break;
            }
        }
        if(c=='2'){
            if(arr==NULL){
                printf("Вы хотите отсортировать несуществующий массив\n");
            }
            else{
                Sort_array(&arr, &len);
                if (arr==NULL && len!=0){
                    break;
                }
            }
        }
        if(c=='3'){
            if (arr==NULL){
                printf("массив не инициализирован\n");
            }
            else{
                Output_array(arr, len);
                if (arr==NULL && len!=0){
                    break;
                }
            }
        }
        if(c=='4'){
            break;
        }
        Wind(&c);
    }
    if(len!=0){
        for (int i=0; i<len; i++){
            free(arr[i].string);
        }
        free(arr);
        len=0;
    }
    return 0;
}
