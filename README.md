#Lab5-TiMP

[![Coverage Status](https://coveralls.io/repos/github/Sxeigumen/Lab5-TiMP/badge.svg?branch=Master)](https://coveralls.io/github/Sxeigumen/Lab5-TiMP?branch=Master)

#1. Создайте CMakeList.txt для библиотеки banking.
CMakeLists.txt для библиотеки banking.
```
cmake_minimum_required(VERSION 3.16.3)
project(banking)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(banking STATIC Account.cpp Account.h Transaction.cpp Transaction.h)
```
CMakeLists.txt для сборки проекта.
```
cmake_minimum_required(VERSION 3.4)

SET(COVERAGE OFF CACHE BOOL "Coverage")
SET(CMAKE_CXX_COMPILER "/usr/bin/g++")

project(TestRunning)

add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/third-party/gtest" "gtest")

#add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/third-party/gtest/gmock gmock)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/banking)

add_executable(RunTest ${CMAKE_CURRENT_SOURCE_DIR}/tests/test.cpp)

if (COVERAGE)
    target_compile_options(RunTest PRIVATE --coverage)
    target_link_libraries(RunTest PRIVATE --coverage)
endif()

target_include_directories(RunTest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/banking)

target_link_libraries(RunTest PRIVATE gtest gtest_main gmock_main banking)
```
#2. Создайте модульные тесты на классы Transaction и Account.
## Используйте mock-объекты.
```
#include <iostream>
#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>
#include <gmock/gmock.h>



struct MockAcc : public Account{
    MockAcc(int id, int balance) : Account(id, balance){}
    MOCK_METHOD1(ChangeBalance, void(int diff));
    MOCK_METHOD0(GetBalance, int());
    MOCK_METHOD0(id, int());
    MOCK_METHOD0(Lock, void());
    MOCK_METHOD0(Unlock, void());
};

struct MockTrans : public Transaction{
    MOCK_METHOD3(Make, bool(Account& from, Account& to, int sum));
    MOCK_METHOD1(set_fee, void(int fee));
    MOCK_METHOD0(fee, int());
};

TEST(Account, AccMockTest1){
    MockAcc BankAccount(1, 15000);
    EXPECT_CALL(BankAccount, ChangeBalance(testing::_)).Times(1);
    EXPECT_CALL(BankAccount, GetBalance()).Times(2);
    EXPECT_CALL(BankAccount, id()).Times(2);
    EXPECT_CALL(BankAccount, Lock()).Times(1);
    BankAccount.GetBalance();
    BankAccount.id();
    BankAccount.Lock();
    BankAccount.ChangeBalance(5000);
    BankAccount.GetBalance();
    BankAccount.id();
}

TEST(Account, AccMockTest2){
    MockAcc BankAccount2(2, 100000);
    EXPECT_CALL(BankAccount2, Lock()).Times(1);
    EXPECT_CALL(BankAccount2, GetBalance()).Times(2);
    EXPECT_CALL(BankAccount2, ChangeBalance(testing::_)).Times(2);
    EXPECT_CALL(BankAccount2, Unlock()).Times(1);
    BankAccount2.Lock();
    BankAccount2.GetBalance();
    BankAccount2.ChangeBalance(100000);
    BankAccount2.GetBalance();
    BankAccount2.Unlock();
    BankAccount2.ChangeBalance(1000);
}


TEST(Account, GetBalance_and_ID){
    Account BankAccount(1, 15000);

    EXPECT_EQ(BankAccount.GetBalance(), 15000);
    EXPECT_EQ(BankAccount.id(), 1);

    BankAccount.Lock();
    BankAccount.ChangeBalance(5000);

    EXPECT_EQ(BankAccount.GetBalance(), 20000);
    EXPECT_EQ(BankAccount.id(), 1);
}

TEST(Account, ChangeBalance_and_Lock){
    Account BankAccount2(2, 100000);
    BankAccount2.Lock();

    EXPECT_EQ(BankAccount2.GetBalance(), 100000);

    BankAccount2.ChangeBalance(100000);

    EXPECT_EQ(BankAccount2.GetBalance(), 200000);

    BankAccount2.Unlock();
    ASSERT_ANY_THROW(BankAccount2.ChangeBalance(1000));
}

TEST(Transaction, TransMockTest1){
    MockTrans Acc_Trans1;
    MockAcc Human1(1, 10000);
    MockAcc Human2(2, 230000);
    MockAcc Car_Shop(3, 8000000);
    MockAcc Organisation(4, 5500);
    EXPECT_CALL(Acc_Trans1, set_fee(testing::_)).Times(1);
    EXPECT_CALL(Acc_Trans1, Make(testing::_, testing::_, testing::_)).Times(2);
    EXPECT_CALL(Acc_Trans1, fee()).Times(1);
    EXPECT_CALL(Human1, GetBalance()).Times(1);
    EXPECT_CALL(Human2, GetBalance()).Times(1);
    Acc_Trans1.set_fee(500);
    Acc_Trans1.Make(Human1, Human2, 2000);
    Acc_Trans1.fee();
    Human1.GetBalance();
    Human2.GetBalance();
    Acc_Trans1.Make(Organisation, Human2, 20000);
}

TEST(Transaction, TransMockTest2){
    MockTrans Acc_Trans2;
    MockAcc Human1(1, 10000);
    MockAcc Human2(2, 230000);
    MockAcc Car_Shop(3, 8000000);
    MockAcc Organisation(4, 5500);
    EXPECT_CALL(Acc_Trans2, Make(testing::_, testing::_, testing::_)).Times(3);
    Acc_Trans2.Make(Car_Shop, Car_Shop, 100000);
    Acc_Trans2.Make(Car_Shop, Organisation, -5600);
    Acc_Trans2.Make(Human2, Car_Shop, 55);
}

TEST(Transaction, Make){
    Transaction Acc_Trans;

    Account Human1(1, 10000);
    Account Human2(2, 230000);
    Account Car_Shop(3, 8000000);
    Account Organisation(4, 5500);

    Acc_Trans.set_fee(500);
    Acc_Trans.Make(Human1, Human2, 2000);

    EXPECT_NE(Acc_Trans.fee(), 350);
    EXPECT_EQ(Human1.GetBalance(), 7500);
    EXPECT_EQ(Human2.GetBalance(), 232000);
    ASSERT_EQ(Acc_Trans.Make(Organisation, Human2, 20000), false);
}

TEST(Transaction, MakeExceptions){
    Transaction Acc_Trans;

    Account Human1(1, 10000);
    Account Human2(2, 230000);
    Account Car_Shop(3, 8000000);
    Account Organisation(4, 5500);

    ASSERT_ANY_THROW(Acc_Trans.Make(Car_Shop, Car_Shop, 100000));
    ASSERT_ANY_THROW(Acc_Trans.Make(Car_Shop, Organisation, -5600));
    ASSERT_ANY_THROW(Acc_Trans.Make(Human2, Car_Shop, 55));
}
```

#3. Сборка на GitHub Action.
```
name: Banking

on:
  push:
      branches: 
      - Master
      
jobs:

  BuildProject:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2

        - name: build banking
          shell: bash
          run: |
            cd banking
            cmake -H. -B_built
            cmake --build _built
          
  Testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: update
        run:  |
          git submodule update --init
          sudo apt install lcov
          sudo apt install g++-7
      
      - name: test
        shell: bash
        run: |
          mkdir _built && cd _built
          CXX=/usr/bin/g++-7 cmake -DCOVERAGE=1 ..
          cmake --build .
          ./RunTest
          lcov -t "banking" -o lcov.info -c -d .
          
      - name: CovBeg
        uses: coverallsapp/github-action@master
        with:
          github-token: ${{ secrets.github_token }}
          parallel: true
          path-to-lcov: ./_built/lcov.info
          coveralls-endpoint: https://coveralls.io

      - name: CovFin
        uses: coverallsapp/github-action@master
        with:
          github-token: ${{ secrets.github_token }}
          parallel-finished: true
```          
  
