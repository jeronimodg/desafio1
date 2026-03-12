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
