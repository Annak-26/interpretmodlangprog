#include <iostream>
#include <string>
#include <vector>
#include <cstring>
#include <string>
#include <stack>
#include <ctype.h>
#include <algorithm>

using namespace std;

enum type_of_lex {
    LEX_NULL,                                               //   0

    LEX_AND, LEX_OR, LEX_NOT, LEX_IF, LEX_ELSE, LEX_DO,     //   6
    LEX_WHILE, LEX_FOR, LEX_BREAK, LEX_GOTO, LEX_PROGRAM,   //  11
    LEX_FALSE, LEX_TRUE, LEX_READ, LEX_WRITE,               //  15
    LEX_INT, LEX_BOOL, LEX_STR, LEX_STRUCT, LEX_REAL,       //  20
    LEX_SEMICOLON, LEX_COMMA, LEX_COLON, LEX_NEQ,           //  24
    LEX_ASSIGN, LEX_LPAREN, LEX_RPAREN, LEX_EQ, LEX_LSS,    //  29
    LEX_GTR, LEX_PLUS, LEX_MINUS, LEX_TIMES, LEX_SLASH,     //  34
    LEX_LEQ, LEX_GEQ, LEX_PERCENT, LEX_LBRACE, LEX_RBRACE, LEX_DOT,  //  40

    LEX_NUM, LEX_ID, LEX_FIN,                                //  43
    POLIZ_LABEL,                                            //  44
    POLIZ_ADDRESS,                                          //  45
    POLIZ_GO,                                               //  46
    POLIZ_FGO                                               //  47
};

class Lex {
private:
    type_of_lex type;
    int value;
    string str_value;
public:
    Lex(type_of_lex t = LEX_NULL, int v = 0, string s = "-") : type(t), value(v), str_value(s) {}
    void put_type(type_of_lex t) { type = t; }
    void put_value(int v) { value = v; }
    void put_string(string s) { str_value = s; }
    type_of_lex get_type() const { return type; }
    int get_value() const { return value; }
    string get_string() const { return str_value; }
    friend ostream& operator << (ostream& s, Lex l);
};

class Ident {
private:
    string name;
    bool declare;
    type_of_lex type;
    bool assign;
    int value;
    string str;
public:
    Ident() {
        declare = false;
        assign = false;
    }
    Ident(const string newname) {
        name = newname;
        declare = false;
        assign = false;
    }
    bool operator== (const string& s) const { return name == s; }
    string get_name() const { return name; }
    string get_string() const { return str; }
    bool get_declare() const { return declare; }
    void put_name(string newname) { name = newname; }
    void put_string(string s) { str = s; }
    void put_declare() { declare = true; }
    type_of_lex get_type() const { return type; }
    void put_type(type_of_lex t) { type = t; }
    bool get_assign() const { return assign; }
    void put_assign() { assign = true; }
    int get_value() const { return value; }
    void put_value(int val) { value = val; }
};

vector<Ident> TID;

int put(const string& buf) {
    for (int i = 0; i < TID.size(); i++) {
        if (TID[i] == buf) return i;
    }
    TID.push_back(Ident(buf));
    return TID.size() - 1;
}

class Scanner {
private:
    FILE* fp;
    char c;
    int look(const string buf, const char** list) {
        for (int i = 0;list[i];i++) {
            if (buf == list[i]) { return i; }
        }
        return 0;
    }
    void gc() { c = fgetc(fp); }
public:
    static const char* TW[], * TD[];
    Scanner(const char* program) {
        fp = fopen(program, "r");
        if (!fp) throw "no open file";
    }
    Lex get_lex();
    ~Scanner() { fclose(fp); }
};

const char* Scanner::TW[] = {
    "", "and", "or", "not", "if", "else", "do", "while", "for",
    "break", "goto", "program", "false", "true", "read", "write",
    "int", "bool", "string", "struct", "real",
    NULL
};

const char* Scanner::TD[] = {
    "", ";", ",", ":", "!=", "=", "(", ")", "==", "<",
    ">", "+", "-", "*", "/", "<=", ">=", "%", "{", "}", ".",
    NULL
};

ostream& operator<< (ostream& s, Lex l) {
    string n;
    if (l.type == LEX_TRUE) n = "true";
    else if (l.type == LEX_FALSE) n = "false";
    else if (l.type <= LEX_REAL) n = Scanner::TW[l.type];
    else if (l.type >= LEX_SEMICOLON && l.type <= LEX_RBRACE)
        n = Scanner::TD[l.type - LEX_SEMICOLON + 1];
    else if (l.type == LEX_NUM) n = "number";
    else if (l.type == LEX_ID) n = TID[l.value].get_name();
    else if (l.type == LEX_FIN) n = "@";
    else if (l.type == POLIZ_LABEL) n = "label";
    else if (l.type == POLIZ_ADDRESS) n = "Addr";
    else if (l.type == POLIZ_GO) n = "!";
    else if (l.type == POLIZ_FGO) n = "!F";
    else
        throw l;
    s << "lex: " << n << ", value: " << l.value << endl;
    return s;
}

Lex Scanner::get_lex() {
    enum state { H, IDENT, NUMB, STRLIT, SLASH, COM, COMS, ALE, NEQ };
    state CS = H;
    string buf;
    int d, j;
    do {
        gc();
        switch (CS) {
        case H:
            if (isspace(c)) {}
            else if (isalpha(c)) {
                buf.push_back(c);
                CS = IDENT;
            }
            else if (isdigit(c)) {
                d = c - '0';
                CS = NUMB;
            }
            else if (c == '/') {
                CS = SLASH;
            }
            else if (c == '=' || c == '<' || c == '>') {
                buf.push_back(c);
                CS = ALE;
            }
            else if (c == EOF) return Lex(LEX_FIN, 0);
            else if (c == '!') {
                buf.push_back(c);
                CS = NEQ;
            }
            else if (c == '"') {
                CS = STRLIT;
            }
            else {
                buf.push_back(c);
                if ((j = look(buf, TD)))
                    return Lex((type_of_lex)(j + (int)LEX_SEMICOLON - 1), j);
                else throw c;
            }
            break;

        case IDENT:
            if (isalpha(c) || isdigit(c) || c == '.') buf.push_back(c);
            else {
                ungetc(c, fp);
                if ((j = look(buf, TW))) {
                    if ((type_of_lex)(j) == LEX_TRUE)
                        return Lex((type_of_lex)(j), 1);
                    else if ((type_of_lex)(j) == LEX_FALSE)
                        return Lex((type_of_lex)(j), 0);
                    return Lex((type_of_lex)(j), j);
                } else
                    return Lex(LEX_ID, put(buf));
            }
            break;

        case NUMB:
            if (isdigit(c)) d = d * 10 + (c - '0');
            else {
                ungetc(c, fp);
                return Lex(LEX_NUM, d);
            }
            break;

        case STRLIT:
            if (c == '"') return Lex(LEX_STR, 0, buf);
            else if (c == EOF) throw c;
            else buf.push_back(c);
            break;

        case SLASH:
            if (c == '*') CS = COMS;
            else if (c == '/') CS = COM;
            else {
                ungetc(c, fp);
                return Lex(LEX_SLASH, look("/", TD));
            }
            break;

        case COMS:
            if (c == '*') {
                while (c == '*') gc();
                if (c == '/') CS = H;
            }
            else if (c == EOF) throw c;
            break;

        case COM:
            if (c == '\n' || c == EOF) CS = H;
            break;

        case ALE:
            if (c == '=') {
                buf.push_back(c);
                if ((j = look(buf, TD)))
                    return Lex((type_of_lex)(j + (int)LEX_SEMICOLON - 1), j);
            }
            else {
                ungetc(c, fp);
                if ((j = look(buf, TD)))
                    return Lex((type_of_lex)(j + (int)LEX_SEMICOLON - 1), j);
            }
            break;

        case NEQ:
            if (c == '=') {
                buf.push_back(c);
                j = look(buf, TD);
                return Lex(LEX_NEQ, j);
            }
            else throw c;
            break;
        }
    } while (true);
}

class strct {
public:
    string type;
    string name;
    vector<Ident> idents;//polya
};

vector<int> labels;
vector<int> gotos;
vector<int> adress;
vector<strct> structs;

stack<int> st_int;
stack<type_of_lex> st_lex;

int find_goto(int n, int label) {
    for (int i = 0; i < gotos.size(); i++)
        if (gotos[i] == n) return adress[i];
    if (label != 0) {
        gotos.push_back(n);
        adress.push_back(label);
        return 0;
    }
    return 0;
}

void label_definder(int n) {
    for (int i = 0; i < labels.size(); i++)
        if (labels[i] == n) throw "redefind";
    labels.push_back(n);
}

type_of_lex from_stack(stack<type_of_lex>& st) {
    type_of_lex t = st.top();
    st.pop();
    return t;
}
Lex from_stack(stack<Lex>& st) {
    Lex t = st.top();
    st.pop();
    return t;
}
int int_from_stack() {
    int t = st_int.top();
    st_int.pop();
    return t;
}

class Parser {
    Scanner scan;

    Lex cl_lex;
    type_of_lex cl_type;
    int cl_val;
    void gl() {
        cl_lex = scan.get_lex();
        cl_type = cl_lex.get_type();
        cl_val = cl_lex.get_value();
        cout << cl_lex;
    }

    bool cycle = false;
    bool breaks = false;
    vector<strct> buf; // буфер для структуры, у которой пока нет имени
    int struct_buf(string& type1) { // ищем структуру данного типа c именемм
        for (int i = 0; i < buf.size(); i++)
            if (buf[i].type == type1) return i;
        return -1;
    }

    type_of_lex type;
    type_of_lex right_type;


    void A(); // program
    void B(); // структура объяление
    void C(); // для объявления переменных
    void D(); // просто переменная или присваивание
    void E(); // иницилизация
    void F(); // проверка тела
    void G(); // правильно ли написано условия в выражениях
    void H(); // Or
    void I(); // And
    void J(); // == < > <= >= !=
    void K(); // + -
    void L(); // * / %
    void M(); // Not () проверяет булевские ли операнды
    void N(); // - +  true false

    void if_func();
    void while_func();
    void for_func();
    void break_func();
    void goto_func();
    void read_func();
    void write_func();

    void dec(type_of_lex type);//проверяет объявлена ли переменная второй раз
    void check_id_in_read();//можно ли ввести
    void check_id();//объявлена ли переменная
    void check_init(type_of_lex type1, type_of_lex type2);//совпадают ли типы при инпицилизации
    void check_op();//проверяем совпадают ли типы для +,-,or,and
    void check_not();//проверяем bool ли переменная
    void eq_type();//одинаковые ли типы при =
    void eq_bool();// проверяем bool ли выражение

public:
    Parser(const char* program) : scan(program) {}
    void parse();
    vector<Lex> poliz;
};

void Parser::parse() {
    gl(); A();
    bool lb = true;//проверяет все ли метки присутствуют
    bool ok;
    int i = 0;

    while (lb && i < gotos.size()) {//есть goto но нет метки
        ok = true;
        int j = 0;
        while (ok && j < labels.size())
            if (gotos[i] == labels[j++]) ok = false;
        i++;
        if (ok) lb = false;
    }

    if (lb == 0) throw "missing label";
    if (cl_type != LEX_FIN) throw "end of file";
    cout << endl << endl << "------- POLIZ -------" << endl << endl;
    for (int i = 0; i < poliz.size(); i++) cout << i << " " << poliz[i];
}

void Parser::if_func() {
    //cout << "execute if" << endl;
    int addr1, addr2;
    gl();

    if (cl_type == LEX_LPAREN) gl(); else throw "missing (";

    G();
    eq_bool();
    addr1 = poliz.size();
    poliz.push_back(Lex());
    poliz.push_back(Lex(POLIZ_FGO));

    if (cl_type == LEX_RPAREN) gl(); else throw "missing )";

    F();
    if (cl_type == LEX_ELSE) {
        addr2 = poliz.size();
        poliz.push_back(Lex());
        poliz.push_back(Lex(POLIZ_GO));
        poliz[addr1] = Lex(POLIZ_LABEL, poliz.size());
        gl(); F();
        poliz[addr2] = Lex(POLIZ_LABEL, poliz.size());
    }
    else
        poliz[addr1] = Lex(POLIZ_LABEL, poliz.size());
}

void Parser::while_func() {
    //cout << "execute while" << endl;
    int addr1, addr2;
    cycle = true;
    addr1 = poliz.size();
    gl();

    if (cl_type == LEX_LPAREN) gl(); else throw "missing (";

    G();
    addr2 = poliz.size();
    poliz.push_back(Lex());
    poliz.push_back(Lex(POLIZ_FGO));
    eq_bool();

    if (cl_type == LEX_RPAREN) gl(); else throw "missing )";

    F();
    if (breaks) {
        breaks = false;
        poliz[addr2] = Lex(POLIZ_LABEL, poliz.size());
        gl();
    } else {
        poliz.push_back(Lex(POLIZ_LABEL, addr1));
        poliz.push_back(Lex(POLIZ_GO));
        poliz[addr2] = Lex(POLIZ_LABEL, poliz.size());
    }

    cycle = false;
}

void Parser::for_func() {
    //cout << "execute for" << endl;
    int addr1, addr2, addr3, addr4;
    cycle = true;
    gl();

    if (cl_type == LEX_LPAREN) gl(); else throw "missing (";

    if (cl_type != LEX_SEMICOLON) {
        G();
        if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
    }
    else gl();
    addr1 = poliz.size();

    if (cl_type != LEX_SEMICOLON) {
        G();
        if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
    }
    else gl();
    addr2 = poliz.size();

    poliz.push_back(Lex());
    poliz.push_back(Lex(POLIZ_FGO));
    addr3 = poliz.size();
    poliz.push_back(Lex());
    poliz.push_back(Lex(POLIZ_GO));
    addr4 = poliz.size();

    if (cl_type != LEX_RPAREN) {
        G();
        if (cl_type == LEX_RPAREN) gl(); else throw "missing )";
    }

    poliz.push_back(Lex(POLIZ_LABEL, addr1));
    poliz.push_back(Lex(POLIZ_GO));
    poliz[addr3] = Lex(POLIZ_LABEL, poliz.size());
    F();

    if (breaks) {
        breaks = false;
        poliz[addr2] = Lex(POLIZ_LABEL, poliz.size());
        gl();
    }
    else {
        poliz.push_back(Lex(POLIZ_LABEL, addr4));
        poliz.push_back(Lex(POLIZ_FGO));
        poliz[addr2] = Lex(POLIZ_LABEL, poliz.size());
    }

    cycle = false;
}

void Parser::break_func() {
    //cout << "execute break" << endl;
    if (cycle) {
        gl();
        if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
        while (cl_type != LEX_RBRACE) gl();
        breaks = true;
    }
    else throw "break not in cycle";
}

void Parser::goto_func() {
    //cout << "execute goto" << endl;
    gl();
    if (cl_type == LEX_ID) {

        int label = find_goto(cl_val, 0);

        if (label) {
            poliz.push_back(Lex(POLIZ_LABEL, label));
            poliz.push_back(Lex(POLIZ_GO));
        }
        else {
            find_goto(cl_val, poliz.size());
            poliz.push_back(Lex());
            poliz.push_back(Lex(POLIZ_GO));
        }
        gl();

        if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
    }
    else throw "goto without label";
}

void Parser::read_func() {
    //cout << "execute read" << endl;
    gl();

    if (cl_type == LEX_LPAREN) gl(); else throw "missing (";

    if (cl_type == LEX_ID) {
        check_id_in_read();
        poliz.push_back(Lex(POLIZ_ADDRESS, cl_val));
        gl();
    }
    else throw "error";
    poliz.push_back(Lex(LEX_READ));

    if (cl_type == LEX_RPAREN) gl(); else throw "missing )";
    if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
}

void Parser::write_func() {
    //cout << "execute write" << endl;
    gl();
    if (cl_type != LEX_LPAREN) throw "missing (";

    gl();
    if (cl_type == LEX_ID) {
        for (int i = 0; i < structs.size(); i++)
            if (structs[i].name == TID[cl_val].get_name())
                throw "writing structure";
    }
    G();
    poliz.push_back(Lex(LEX_WRITE));
    while (cl_type == LEX_COMMA) {
        gl();
        if (cl_type == LEX_ID) {
            for (int i = 0; i < structs.size(); i++)
                if (structs[i].name == TID[cl_val].get_name())
                    throw "writing structure";
        }
        G();
        poliz.push_back(Lex(LEX_WRITE));
    }
    if (cl_type == LEX_RPAREN) gl(); else throw "missing )";
    if (cl_type == LEX_SEMICOLON) gl(); else throw "missing ;";
}

void Parser::E() {
    //cout << "execute E" << endl;
    if (cl_type == LEX_INT || cl_type == LEX_BOOL || cl_type == LEX_STR) {
        type = cl_type;
        gl(); C();
        if (cl_type == LEX_SEMICOLON) gl(); else throw "missing semicolon";
        while (cl_type == LEX_INT || cl_type == LEX_BOOL || cl_type == LEX_STR) {
            type = cl_type;
            gl(); C();
            if (cl_type == LEX_SEMICOLON) gl(); else throw "missing semicolon";
        }
    }
    else if (cl_type == LEX_ID) {
        string s_name;
        string s_type = TID[cl_val].get_name();
        strct str;
        str.idents = buf[struct_buf(s_type)].idents;
        str.type = s_type;
        int tmp;

        gl(); 
        s_name = TID[cl_val].get_name();
        str.name = s_name;
        structs.push_back(str);
        st_int.push(cl_val);
        dec(LEX_STRUCT);
        for (int i = 0; i < str.idents.size(); i++) {
            tmp = put(s_name + '.' + str.idents[i].get_name());
            st_int.push(tmp);
            dec(str.idents[i].get_type());
        }
        gl();
        while (cl_type == LEX_COMMA) {
            gl();
            s_name = TID[cl_val].get_name();
            str.name = s_name;
            structs.push_back(str);
            st_int.push(cl_val);
            dec(LEX_STRUCT);
            for (int i = 0; i < str.idents.size(); i++) {
                tmp = put(s_name + '.' + str.idents[i].get_name());
                st_int.push(tmp);
                dec(str.idents[i].get_type());
            }
            gl();
        }
        if (cl_type == LEX_SEMICOLON) gl(); else throw "';' was expected";
        if (cl_type != LEX_LBRACE) E();
    }
}

void Parser::A() { 
    //cout << "execute A" << endl;
    if (cl_type == LEX_PROGRAM) gl(); else throw "missing program";
    if (cl_type == LEX_LBRACE) gl(); else throw "missing bracket";
    while (cl_type == LEX_STRUCT) { 
        B(); gl();
    }
    E(); 
    while (cl_type != LEX_RBRACE) F();
    if (cl_type == LEX_RBRACE) gl(); else throw "missing bracket";
}

void Parser::B() {
    //cout << "execute B" << endl;
    strct str;
    Ident p;
    gl();
    if (cl_type == LEX_ID) {
        str.type = TID[cl_val].get_name();
        gl();
    } else throw "name error";

    if (cl_type == LEX_LBRACE) gl(); else throw "missing {";

    while (cl_type != LEX_RBRACE) {
        if (cl_type != LEX_INT && cl_type != LEX_BOOL && cl_type != LEX_STR) throw "id error";
        type_of_lex s_type = cl_type;
        gl();
        while (cl_type != LEX_SEMICOLON) {
            if (cl_type == LEX_ID) {
                p.put_name(TID[cl_val].get_name());
                p.put_type(s_type);
                str.idents.push_back(p);
                gl();
            } else throw "id error";
            if (cl_type == LEX_COMMA) gl();
            else if (cl_type != LEX_SEMICOLON) throw "missing ;";
        }
        gl();
    }
    buf.push_back(str);
}

void Parser::C() {
    //cout << "execute C" << endl;
    D();
    while (cl_type == LEX_COMMA) {
        gl(); D();
    }
}

void Parser::D() {
    //cout << "execute D" << endl;
    int tmp;
    if (cl_type == LEX_ID) {
        tmp = cl_val;
        st_int.push(tmp);
        dec(type);
        gl();
    } else throw "error";
    if (cl_type == LEX_ASSIGN) {
        poliz.push_back(Lex(POLIZ_ADDRESS, tmp));
        gl(); N();
        check_init(type, right_type);
    }
}

void Parser::F() {
    //cout << "execute F" << endl;

    if (cl_type == LEX_IF) if_func();
    else if (cl_type == LEX_WHILE) while_func();
    else if (cl_type == LEX_FOR) for_func();
    else if (cl_type == LEX_BREAK) break_func();
    else if (cl_type == LEX_GOTO) goto_func();
    else if (cl_type == LEX_READ) read_func();
    else if (cl_type == LEX_WRITE) write_func();

    else if (cl_type == LEX_LBRACE) {
        gl(); 
        while (cl_type != LEX_RBRACE) F();
        if (cl_type == LEX_RBRACE) gl();
        else throw "missing }";
    }
    else {
        G();
        if (cl_type == LEX_SEMICOLON) {
            poliz.push_back(Lex(LEX_SEMICOLON));
            gl();
        }
        else if (cl_type == LEX_ID) M();
        else if (cl_type != LEX_RBRACE && cl_type != LEX_FIN) throw "missing }";
    }
}

void Parser::G() {
    //cout << "execute G" << endl;
    int addr;
    if (cl_type == LEX_ID) {
        string name = TID[cl_val].get_name();
        int j = -1;
        for (int i = 0; i < structs.size(); i++)
            if (structs[i].name == name) j = i;
        if (j == -1) {
            addr = poliz.size();
            H();
            if (cl_type == LEX_ASSIGN) {
                poliz[addr].put_type(POLIZ_ADDRESS);
                gl(); G();
                eq_type();
            }
        } else {
            string s_type = structs[j].type;
            string s_name = structs[j].name;
            vector<Ident> idents = structs[j].idents;
            gl();
            if (cl_type == LEX_SEMICOLON) return;
            else if (cl_type != LEX_ASSIGN) throw "error operation";
            gl();
            name = TID[cl_val].get_name();
            j = -1;
            for (int i = 0; i < structs.size(); i++)
                if (structs[i].name == name) j = i;
            if (j == -1) throw "error assign";
            if (s_type != structs[j].type) throw "error assign"; // если разные структурные типы
            for (int i = 0; i < idents.size(); i++) {

                string cur_name1 = s_name + '.' + idents[i].get_name();
                string cur_name2 = structs[j].name + '.' + idents[i].get_name();

                auto i1 = find(TID.begin(), TID.end(), cur_name1) - TID.begin();
                auto i2 = find(TID.begin(), TID.end(), cur_name2) - TID.begin();
                
                TID[i1].put_assign();
                poliz.push_back(Lex(POLIZ_ADDRESS, i1));
                st_lex.push(TID[i1].get_type());
                st_lex.push(LEX_ASSIGN);
                poliz.push_back(Lex(LEX_ID, i2));
                st_lex.push(TID[i2].get_type());

                poliz.push_back(Lex(LEX_ASSIGN));
                poliz.push_back(Lex(LEX_SEMICOLON));
            }
            poliz.pop_back();
            gl();
        }
    }
    else H();
}

void Parser::H() {
    //cout << "execute H" << endl;
    I();
    while (cl_type == LEX_OR) {
        st_lex.push(cl_type);
        gl(); I();
        check_op();
    }
}

void Parser::I() {
    //cout << "execute I" << endl;
    J();
    while (cl_type == LEX_AND) {
        st_lex.push(cl_type);
        gl(); J();
        check_op();
    }
}

void Parser::J() {
    //cout << "execute J" << endl;
    K();
    while (cl_type == LEX_EQ || cl_type == LEX_LSS || cl_type == LEX_GTR || cl_type == LEX_LEQ || cl_type == LEX_GEQ || cl_type == LEX_NEQ) {
        st_lex.push(cl_type);
        gl(); K();
        check_op();
    }
}

void Parser::K() {
    //cout << "execute K" << endl;
    L();
    while (cl_type == LEX_PLUS || cl_type == LEX_MINUS) {
        st_lex.push(cl_type);
        gl(); L();
        check_op();
    }
}

void Parser::L() {
    //cout << "execute L" << endl;
    M();
    while (cl_type == LEX_TIMES || cl_type == LEX_SLASH) {
        st_lex.push(cl_type);
        gl(); M();
        check_op();
    }
}

void Parser::M() {
    //cout << "execute M" << endl;
    int addr, lb, gotolb;
    if (cl_type == LEX_NOT) {
        gl(); M();
        check_not();
    } 
    else if (cl_type == LEX_ID) {
        bool isnt_label = TID[cl_val].get_declare();
        if (!isnt_label) {
            lb = cl_val;
            gl();
            if (cl_type == LEX_COLON) {
                label_definder(lb);
                addr = poliz.size();
                gotolb = find_goto(lb, addr);
                if (gotolb) poliz[gotolb] = Lex(POLIZ_LABEL, addr);
                gl(); F();
            } else throw "undeclared id";
        } else {
            poliz.push_back(cl_lex);
            st_lex.push(TID[cl_val].get_type());
            gl();
        }
    }
    else if (cl_type == LEX_LPAREN) {
        gl(); G();
        if (cl_type == LEX_RPAREN) gl(); //else throw "missing )";
    }
    else N();
}

void Parser::N() {
    //cout << "execute N" << endl;
    if (cl_type == LEX_MINUS) {
        gl();
        if (cl_type == LEX_NUM) {
            st_lex.push(cl_type);
            poliz.push_back(Lex(cl_type, -cl_val));
            right_type = cl_type;
            gl();
        } else throw "missing num";
    }

    else if (cl_type == LEX_PLUS) {
        gl();
        if (cl_type == LEX_NUM) {
            st_lex.push(LEX_INT);
            poliz.push_back(cl_lex);
            right_type = cl_type;
            gl();
        } else throw "missing num";
    }

    else if (cl_type == LEX_NUM) {
        st_lex.push(cl_type);
        poliz.push_back(cl_lex);
        right_type = cl_type;
        gl();
    }

    else if (cl_type == LEX_TRUE) {
        st_lex.push(LEX_BOOL);
        poliz.push_back(cl_lex);
        right_type = cl_type;
        gl();
    }

    else if (cl_type == LEX_FALSE) {
        st_lex.push(LEX_BOOL);
        poliz.push_back(cl_lex);
        right_type = cl_type;
        gl();
    }

    else if (cl_type == LEX_STR) {
        st_lex.push(cl_type);
        poliz.push_back(cl_lex);
        right_type = cl_type;
        gl();
    }

    else "error";
}

bool types_are_equal(type_of_lex type1, type_of_lex type2) {
    if (type1 == LEX_INT || type1 == LEX_NUM) {
        if (type2 == LEX_INT || type2 == LEX_NUM) return true;
    } else if (type1 == type2 && type1 == LEX_BOOL) return true;
    else if (type1 == type2 && type1 == LEX_STR) return true;
    return false;
}

void Parser::dec(type_of_lex type) {
    int i;
    while (!st_int.empty())
    {
        i = int_from_stack();
        if (TID[i].get_declare()) throw "already declared";
        else {
            TID[i].put_declare();
            TID[i].put_type(type);
        }
    }
}

void Parser::check_id()
{
    if (TID[cl_val].get_declare()) poliz.push_back(cl_lex);
    else throw "not declared";
}

void Parser::check_init(type_of_lex type1, type_of_lex type2) {
    if (type1 == LEX_BOOL && type2 != LEX_TRUE && type2 != LEX_FALSE && type2 != LEX_BOOL)
        throw "init types error";
    else if (!types_are_equal(type1, type2))
        throw "init types error";

    poliz.push_back(Lex(LEX_ASSIGN));
    poliz.push_back(Lex(LEX_SEMICOLON));
}

void Parser::check_op()
{
    type_of_lex type1, type2, op, t = LEX_INT, r = LEX_BOOL;

    type2 = from_stack(st_lex);
    op = from_stack(st_lex);
    type1 = from_stack(st_lex);

    if (op == LEX_PLUS || op == LEX_MINUS || op == LEX_TIMES || op == LEX_SLASH || op == LEX_PERCENT)
        r = LEX_INT;
    if (op == LEX_PLUS && type1 == LEX_STR) r = t = LEX_STR;
    if (op == LEX_OR || op == LEX_AND) t = LEX_BOOL;
    if (op == LEX_LSS || op == LEX_GTR || op == LEX_EQ || op == LEX_NEQ)
        if (type1 == LEX_INT) t = LEX_INT;
        else t = LEX_STR;

    if (!types_are_equal(type1, type2) && !types_are_equal(type1, t)) throw "op types error";

    st_lex.push(r);
    poliz.push_back(Lex(op));
}

void Parser::check_not()
{
    type_of_lex t = from_stack(st_lex);
    if (t != LEX_BOOL) throw "not types error";
    else {
        st_lex.push(LEX_BOOL);
        poliz.push_back(Lex(LEX_NOT));
    }
}

void Parser::eq_type()
{
    type_of_lex type1 = from_stack(st_lex);
    type_of_lex type2 = from_stack(st_lex);
    if (!types_are_equal(type1, type2)) throw "eq types error";
    poliz.push_back(Lex(LEX_ASSIGN));
}

void Parser::eq_bool()
{
    type_of_lex t = from_stack(st_lex);
    if (t != LEX_BOOL) throw "bool types error";
}

void Parser::check_id_in_read()
{
    if (!TID[cl_val].get_declare()) throw "not declared";
    for (int i = 0; i < structs.size(); i++)
        if (structs[i].name == TID[cl_val].get_name())
            throw "reading structure";
}

class Executer {
public:
    void execute(vector<Lex>& poliz);
};

class Interpretator {
    Parser pars;
    Executer exec;
public:
    Interpretator(const char* program) : pars(program) {}
    void interpretation();
};

void Executer::execute(vector<Lex>& poliz) {

    cout << endl << endl << "------- INTERPRETATOR -------" << endl << endl;
    Lex lex, lex1, lex2;
    stack<Lex> st;
    int i = 0, val;
    while (i < poliz.size()) {
        lex = poliz[i];
        cout << "current " << lex;
        switch (lex.get_type()) {

            case LEX_TRUE:
                st.push(lex);
                break;

            case LEX_FALSE:
                st.push(lex);
                break;

            case LEX_NUM:
                st.push(lex);
                break;

            case LEX_STR:
                st.push(lex);
                break;

            case POLIZ_ADDRESS:
                st.push(lex);
                break;

            case POLIZ_LABEL:
                st.push(lex);
                break;

            case LEX_ID:
                val = lex.get_value();
                if (TID[val].get_assign()) {
                    if (TID[val].get_type() == LEX_STR)
                        st.push(Lex(TID[val].get_type(), 0, TID[val].get_string()));
                    else
                        st.push(Lex(TID[val].get_type(), TID[val].get_value()));
                    break;
                }
                else throw "id error";

            case LEX_NOT:
                lex1 = from_stack(st);
                st.push(Lex(lex1.get_type(), !lex1.get_value()));
                break;

            case LEX_OR:
                lex1 = from_stack(st); 
                lex2 = from_stack(st);
                st.push(Lex(lex1.get_type(), lex1.get_value() || lex2.get_value()));
                break;

            case LEX_AND:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                st.push(Lex(lex1.get_type(), lex1.get_value() && lex2.get_value()));
                break;

            case POLIZ_GO:
                lex1 = from_stack(st);
                i = lex1.get_value() - 1;
                break;

            case POLIZ_FGO:
                lex1 = from_stack(st);
                if (lex1.get_value() != poliz.size()) {
                    lex2 = from_stack(st);
                    if (lex2.get_value() == 0) i = lex1.get_value() - 1;
                } else i = lex1.get_value() - 1;
                break;

            case LEX_WRITE:
                lex1 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    cout << lex1.get_string() << endl;
                else if (lex1.get_type() == LEX_INT || lex1.get_type() == LEX_NUM)
                    cout << lex1.get_value() << endl;
                else {
                    if (lex1.get_value()) cout << "true" << endl;
                    else cout << "false" << endl;
                }
                break;

            case LEX_READ:
                int n;
                lex1 = from_stack(st);
                val = lex1.get_value();
                if (TID[val].get_type() == LEX_INT) {
                    cout << "int " << TID[val].get_name() << " = ";
                    cin >> n;
                    TID[val].put_value(n);
                } else if (TID[val].get_type() == LEX_BOOL) {
                    string s;
                    cout << "bool " << TID[val].get_name() << " = ";
                    cin >> s;
                    if (s == "true") n = 1;
                    else n = 0;
                    TID[val].put_value(n);
                } else {
                    string s;
                    cout << "string " << TID[val].get_name() << " = ";
                    cin >> s;
                    TID[val].put_string(s);
                }
                TID[val].put_assign();
                break;

            case LEX_PLUS:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    st.push(Lex(lex1.get_type(), 0, lex1.get_string() + lex2.get_string()));
                else
                    st.push(Lex(lex1.get_type(), lex1.get_value() + lex2.get_value()));
                break;

            case LEX_TIMES:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                st.push(Lex(lex1.get_type(), lex1.get_value() * lex2.get_value()));
                break;

            case LEX_MINUS:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                st.push(Lex(lex1.get_type(), lex2.get_value() - lex1.get_value()));
                break;

            case LEX_SLASH:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_value() != 0) {
                    st.push(Lex(lex1.get_type(), lex2.get_value() / lex1.get_value()));
                    break;
                } else throw "division on zero";

            case LEX_PERCENT:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_value() != 0) {
                    st.push(Lex(lex1.get_type(), lex2.get_value() % lex1.get_value()));
                    break;
                }
                else throw "division on zero";

            case LEX_EQ:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    st.push(Lex(LEX_BOOL, lex1.get_string() == lex2.get_string()));
                else
                    st.push(Lex(LEX_BOOL, lex1.get_value() == lex2.get_value()));
                break;

            case LEX_LSS:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    st.push(Lex(LEX_BOOL, lex2.get_string() < lex1.get_string()));
                else
                    st.push(Lex(LEX_BOOL, lex2.get_value() < lex1.get_value()));
                break;

            case LEX_GTR:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    st.push(Lex(LEX_BOOL, lex1.get_string() < lex2.get_string()));
                else
                    st.push(Lex(LEX_BOOL, lex1.get_value() < lex2.get_value()));
                break;

            case LEX_LEQ:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                st.push(Lex(LEX_BOOL, lex2.get_value() <= lex1.get_value()));
                break;

            case LEX_GEQ:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                st.push(Lex(LEX_BOOL, lex2.get_value() >= lex1.get_value()));
                break;

            case LEX_NEQ:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR)
                    st.push(Lex(LEX_BOOL, lex1.get_string() != lex2.get_string()));
                else
                    st.push(Lex(LEX_BOOL, lex1.get_value() != lex2.get_value()));
                break;

            case LEX_ASSIGN:
                lex1 = from_stack(st);
                lex2 = from_stack(st);
                if (lex1.get_type() == LEX_STR) {
                    TID[lex2.get_value()].put_string(lex1.get_string());
                    st.push(Lex(POLIZ_ADDRESS, lex2.get_value(), lex2.get_string()));
                } else if (lex1.get_type() == POLIZ_ADDRESS) {
                    TID[lex2.get_value()].put_value(TID[lex1.get_value()].get_value());
                    st.push(Lex(POLIZ_ADDRESS, lex2.get_value(), lex2.get_string()));
                } else {
                    TID[lex2.get_value()].put_value(lex1.get_value());
                    st.push(Lex(POLIZ_ADDRESS, lex2.get_value(), lex2.get_string()));
                }
                TID[lex2.get_value()].put_assign();
                break;

            case LEX_SEMICOLON:
                st.pop();
                break;

            default:
                throw "error";
                break;
            } 
        i += 1;
    };
}

void Interpretator::interpretation() {
    pars.parse();
    exec.execute(pars.poliz);
}

int main(int argc, char* argv[]) {
    cout << endl << endl << "------- LEX SCANNER -------" << endl << endl;
    try {
        Scanner scan(argv[1]);
        Interpretator inter(argv[1]);
        inter.interpretation();
    }
    catch (char c) {
        if (c == EOF) cout << "end of file" << endl;
        else cout << "char" << c << endl;
        return 1;
    }
    catch (const char* str) {
        cout << str << endl;
        return 1;
    }
    catch (Lex l) {
        cout << "lex" << endl;
        return 1;
    }
}
