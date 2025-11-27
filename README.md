#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <chrono>

using namespace std;

const string PATRON = "TGTACCTTACAATCG";

// ------------------------------------------
// TRIM → elimina espacios, tab, saltos de línea
// ------------------------------------------
string trim(const string& s) {
    size_t i = s.find_first_not_of(" \t\r\n");
    if (i == string::npos) return "";
    size_t j = s.find_last_not_of(" \t\r\n");
    return s.substr(i, j - i + 1);
}

// ------------------------------------------
// Valida ADN
// ------------------------------------------
bool esADNValido(const string& s) {
    if (s.empty()) return false;
    for (char c : s) {
        char u = toupper(c);
        if (u != 'A' && u != 'T' && u != 'C' && u != 'G') return false;
    }
    return true;
}

// ------------------------------------------
// Construye tabla LPS de KMP
// ------------------------------------------
void construirLPS(const string& p, vector<int>& lps) {
    int j = 0;
    lps[0] = 0;

    for (int i = 1; i < p.size(); i++) {
        while (j > 0 && p[i] != p[j]) {
            j = lps[j - 1];
        }
        if (p[i] == p[j]) j++;
        lps[i] = j;
    }
}

// ------------------------------------------
// KMP_count → cuántas veces aparece el patron
// ------------------------------------------
int KMP_count(const string& texto, const string& patron) {
    int n = texto.size();
    int m = patron.size();
    if (m == 0) return 0;

    vector<int> lps(m);
    construirLPS(patron, lps);

    int i = 0, j = 0, count = 0;

    while (i < n) {
        if (texto[i] == patron[j]) {
            i++; j++;
        }
        if (j == m) {
            count++;
            j = lps[j - 1]; 
        }
        else if (i < n && texto[i] != patron[j]) {
            if (j != 0) j = lps[j - 1];
            else i++;
        }
    }
    return count;
}

// ------------------------------------------
// PROGRAMA PRINCIPAL
// ------------------------------------------
int main() {

    string ruta = "C:\\Users\\Pc\\Documents\\DATASET\\dataset_names_sequences.txt";
    ifstream file(ruta);

    if (!file.is_open()) {
        cout << "No se pudo abrir el archivo.\n";
        return 0;
    }

    int validas = 0, invalidas = 0, vacias = 0;
    long long totalCoincidencias = 0;

    string linea;

    auto t0 = chrono::high_resolution_clock::now();

    while (getline(file, linea)) {

        linea = trim(linea);

        if (linea.empty()) {
            vacias++;
            continue;
        }

        // Extraer secuencia después de ; o ,
        size_t pos = linea.find(';');
        if (pos == string::npos) pos = linea.find(',');

        string sec = (pos != string::npos) ? linea.substr(pos + 1) : linea;
        sec = trim(sec);

        if (!esADNValido(sec)) {
            invalidas++;
            continue;
        }

        validas++;

        totalCoincidencias += KMP_count(sec, PATRON);
    }

    auto tf = chrono::high_resolution_clock::now();
    long long tiempoTotal = chrono::duration_cast<chrono::microseconds>(tf - t0).count();

    // ------------------------------------------
    // CADENA MANUAL
    // ------------------------------------------
    string cadManual;
    cout << "\nIngresa una cadena manual: ";
    cin >> cadManual;

    cadManual = trim(cadManual);

    if (esADNValido(cadManual)) {
        int rep = KMP_count(cadManual, PATRON);
        cout << "\nEl patrón aparece " << rep << " veces en tu cadena.\n";
        totalCoincidencias += rep;
    }
    else {
        cout << "\nCadena ingresada inválida.\n";
    }

    // ------------------------------------------
    // RESULTADOS
    // ------------------------------------------
    cout << "\n===== RESULTADOS =====\n";
    cout << "Cadenas validas:   " << validas << "\n";
    cout << "Cadenas invalidas: " << invalidas << "\n";
    cout << "Cadenas vacias:    " << vacias << "\n";
    cout << "Coincidencias totales del patrón: " << totalCoincidencias << "\n";
    cout << "Tiempo total (microsegundos): " << tiempoTotal << "\n";

    return 0;
}
