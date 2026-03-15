#include <iostream>
using namespace std;
void imprimir(bool **matriz, unsigned short int alt, unsigned short int anch){
    for (unsigned short int i=0;i<alt;i++){
        cout << "|";
        for (unsigned short int j=0;j<anch;j++){
            if (matriz[i][j]==false){
                cout << " . ";
            }
            else{
                cout << "[]";
            }
        }
        cout << "|" << endl;
    }
}
int  main(){
    unsigned short int ancho, alto;
    //unsigned short int prueba[4][2]={{5,6},{4,5},{3,4},{2,3}};
    do{
        cout << "ingrese el ancho de la base que debe ser multiplo de 8: ";
        cin >> ancho;
        cout << "ingrese la altura de la matriz que debe ser minimo de 8: ";
        cin >> alto;
    }while(ancho%8!=0 || ancho<8 || alto<8);
    cout << "felicidades"<< endl;
    bool **matriz = new bool*[alto];
    for (unsigned short int i=0;i<alto;i++){
        matriz[i] = new bool[ancho];
    }
    for(unsigned short int j=0;j<alto*ancho;j++){
        *(*matriz+j)=false;
    }
    /*for(int o=0;o<4;o++){
        for(int r=0;r<1;r++){
            matriz[prueba[o][r]][prueba[o][r+1]]=true;
        }
    }*/
    imprimir(matriz, alto, ancho);
    for (unsigned short int k=0;k<alto;k++){
        delete[] matriz[k];
    }
    delete[] matriz;
    return 0;
}
