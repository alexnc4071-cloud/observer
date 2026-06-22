#include <iostream>
#include <fstream>
#include <vector>
#include <typeinfo>

class Observer {
 public:
  virtual void onWarning(const std::string& message) {}
  virtual void onError(const std::string& message) {}
  virtual void onFatalError(const std::string& message) {}
  virtual ~Observer() = default;
};

class Warning : public Observer {
 public:
  void onWarning(const std::string& message) override {
    std::cout << message << std::endl;
  }
};

class Error : public Observer {
 public:
  std::ofstream file_;
  std::string path_;

  Error(const std::string path) : path_(path) {}

  void onError(const std::string& message) override {
    file_.open(path_);
    if (file_.is_open()) {
      file_ << message << std::endl;
      file_.close();
    } else {
      std::cout << "Class Error, file is not open\n";
    }
  }
};

class FatalError : public Observer {
 public:
  std::ofstream file_;
  std::string path_;

  FatalError(const std::string path) : path_(path) {}

  void onFatalError(const std::string& message) override {
    file_.open(path_);
    if (file_.is_open()) {
      file_ << message << std::endl;
      file_.close();
      std::cout << message << std::endl;
    } else {
      std::cout << "Class FatalError, file is not open\n";
    }
  }
};

class MyClass {

 public:
   
 std::vector<Observer*> vec_;

 void addObserver(Observer* obj) {

     vec_.push_back(obj);
 }

 void removeObserver(Observer * obj) { 
     auto it = std::remove(vec_.begin(), vec_.end(), obj);
     vec_.erase(it, vec_.end());
 }

  void warning(const std::string& message) const {
   for (auto& obj : vec_) {
     if (typeid(*obj) == typeid(Warning)) {
       obj->onWarning(message);
     }
   }
 }

   void fatalError(const std::string& message) const {
   for (auto& obj : vec_) {
     if (typeid(*obj) == typeid(FatalError)) {
       obj->onFatalError(message);
     }
   }
 }

    void error(const std::string& message) const {
   for (auto& obj : vec_) {
     if (typeid(*obj) == typeid(Error)) {
       obj->onError(message);
     }
   }
 }

};

int main() { 

    MyClass my_obj;
    Warning war_obj;
    Error error_obj("text.txt");
    FatalError fatal_error_obj("text1.txt");

    my_obj.addObserver(&war_obj);
    my_obj.addObserver(&error_obj);
    my_obj.addObserver(&fatal_error_obj);
    my_obj.error("Hello error\n");
    my_obj.fatalError("Hello fatal_error\n");
    my_obj.warning("Hello Warning\n");
   
    my_obj.removeObserver(&fatal_error_obj);
    my_obj.error("Hello error\n");
    my_obj.fatalError("Hello fatal_error\n");
    my_obj.warning("Hello Warning\n");

}
