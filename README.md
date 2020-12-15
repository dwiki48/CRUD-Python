﻿# CRUD-Python pyMysql

    from PyQt5 import QtCore, QtGui, QtWidgets
    import pymysql
    import random


    class Ui_MainWindow(object):
        def koneksi(self):
            pymysql.connect(db='db_pyojol', user='root', passwd='',
                            host='localhost', port=3306, autocommit=True)

        def messagebox(self, title, message):
            mess = QtWidgets.QMessageBox()
            mess.setWindowTitle(title)
            mess.setText(message)
            mess.setStandardButtons(QtWidgets.QMessageBox.Ok)
            mess.exec_()

        def save(self):
            # input
            nama = self.namaCostumerLineEdit.text()
            idcus = random.randint(0, 50)
            nope = self.nomorTeleponLineEdit.text()
            jtempuh = self.jarakTempuhKmLineEdit.text()
            harga = int(jtempuh) * 10000

            # koneksi database
            con = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                  host='localhost', port=3306, autocommit=True)

            # cek nama
            koneksi = con.cursor()
            ceknama = "SELECT nama FROM customer WHERE nama='"+str(nama)+"'"
            koneksi.execute(
                "SELECT id_customer FROM customer WHERE nama='"+str(nama)+"'")
            cekid = koneksi.fetchone()
            dataNama = koneksi.execute(ceknama)

            # validasi input nama
            if dataNama == 0:
                # cek potongan
                harga1 = harga * 0.05
                harga2 = harga * 0.10
                total = harga - harga1 if int(jtempuh) >= 50 else harga
                total1 = harga - harga2 if int(jtempuh) >= 100 else harga
                insert = (nama, idcus, nope, jtempuh, 0, total)
                insert1 = (nama, idcus, nope, jtempuh, harga1, total)
                insert2 = (nama, idcus, nope, jtempuh, harga2, total1)
                hasil = insert1 if int(jtempuh) >= 50 else insert
                hasil1 = insert2 if int(jtempuh) >= 100 else insert1
                diskon = hasil if int(jtempuh) <= 100 else hasil1

            elif dataNama <= 2:
                # potongan nama = 2
                idcussama = cekid[0]
                potongan1 = harga * 0.20
                diskonNama = harga - potongan1
                harga1 = diskonNama * 0.05
                hargaTot = potongan1 + harga1
                harga2 = potongan1 * 0.10
                hargaTot1 = diskonNama + harga2

                total = diskonNama - harga1 if int(jtempuh) >= 50 else diskonNama
                total1 = diskonNama - harga2 if int(jtempuh) >= 100 else diskonNama

                insert = (nama, idcussama, nope, jtempuh, potongan1, total)
                insert1 = (nama, idcussama, nope, jtempuh, hargaTot, total)
                insert2 = (nama, idcussama, nope, jtempuh, hargaTot1, total1)
                hasil = insert1 if int(jtempuh) >= 50 else insert
                hasil1 = insert2 if int(jtempuh) >= 100 else insert1
                diskon = hasil if int(jtempuh) <= 100 else hasil1

            elif dataNama >= 5:
                # potongan nama = 5
                potongan1 = harga * 0.30
                diskonNama = harga - potongan1
                ppn = diskonNama + diskonNama * 0.10
                harga1 = ppn * 0.05
                hargaTot = potongan1 + harga1
                harga2 = potongan1 * 0.10
                hargaTot1 = ppn + harga2

                total = ppn - harga1 if int(jtempuh) >= 50 else ppn
                total1 = ppn - harga2 if int(jtempuh) >= 100 else ppn

                insert = (nama, idcus, nope, jtempuh, potongan1, total)
                insert1 = (nama, idcus, nope, jtempuh, hargaTot, total)
                insert2 = (nama, idcus, nope, jtempuh, hargaTot1, total1)
                hasil = insert1 if int(jtempuh) >= 50 else insert
                hasil1 = insert2 if int(jtempuh) >= 100 else insert1
                diskon = hasil if int(jtempuh) <= 100 else hasil1

            # validasi input
            if nama.isalpha():
                try:
                    int(nope)
                    int(jtempuh)
                    cur = con.cursor()
                    sql = "INSERT INTO customer(nama, id_customer, nope, jtempuh, potongan, harga)" + \
                        "VALUES"+str(diskon)
                    cur.execute(sql)
                    self.messagebox("SUKSES", "Data Tersimpan")
                except ValueError:
                    self.messagebox("GAGAL", "Data Gagal Tersimpan")
            if not nama.isalpha():
                self.messagebox("GAGAL", "Data Gagal Tersimpan")

        def tampil(self):
            db = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                 host='localhost', port=3306, autocommit=True)
            cursor = db.cursor()
            cursor.execute(
                "SELECT nama, id_customer, nope, jtempuh, potongan, harga FROM customer")
            query = cursor.fetchall()
            self.tableWidget.setRowCount(0)
            for row_number, row_data in enumerate(query):
                self.tableWidget.insertRow(row_number)
                for column_number, data in enumerate(row_data):
                    self.tableWidget.setItem(
                        row_number, column_number, QtWidgets.QTableWidgetItem(str(data)))

        def update(self):
            nama = self.namaCostumerLineEdit.text()
            idcus = self.idCostumerAutoLineEdit.text()
            nope = self.nomorTeleponLineEdit.text()
            jtempuh = self.jarakTempuhKmLineEdit.text()
            harga = int(jtempuh) * 10000
            harga1 = harga * 0.05
            harga2 = harga * 0.10
            potongan = harga1 if int(jtempuh) >= 50 else 0
            potongan1 = harga2 if int(jtempuh) >= 100 else 0
            totPot = potongan if int(jtempuh) <= 100 else potongan1

            total = 0
            if int(jtempuh) >= 50:
                total = harga-harga1
            elif int(jtempuh) >= 100:
                total = harga-harga1

            con = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                  host='localhost', port=3306, autocommit=True)
            cur = con.cursor()
            sql = cur.execute(
                "UPDATE customer SET nama=%s, nope=%s, jtempuh=%s, potongan=%s, harga=%s WHERE id_customer=%s", (nama, nope, jtempuh, totPot, total, idcus))
            if (sql):
                self.messagebox("SUKSES", "Data Berhasil Di Update")
            else:
                self.messagebox("GAGAL", "Data Gagal Di Update")

        def delete(self):
            nama = self.namaCostumerLineEdit.text()
            con = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                  host='localhost', port=3306, autocommit=True)
            cur = con.cursor()
            sql = "DELETE FROM customer where nama=%s"
            data = cur.execute(sql, (nama))
            if (data):
                self.messagebox("SUKSES", "Data Berhasil Di HAPUS")
            else:
                self.messagebox("GAGAL", "Data GAGAL Di HAPUS")

        def form(self):
            nama = self.namaCostumerLineEdit.text()
            nope = self.nomorTeleponLineEdit.text()
            db = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                 host='localhost', port=3306, autocommit=True)
            cursor = db.cursor()
            cursor.execute(
                "SELECT nama, id_customer, nope, jtempuh, potongan, harga FROM customer WHERE nama='"+str(nama)+"' OR nope='"+str(nope)+"'")
            data = cursor.fetchall()
            if (data):
                for tp in data:
                    self.namaCostumerLineEdit.setText("" + str(tp[0]))
                    self.idCostumerAutoLineEdit.setText("" + str(tp[1]))
                    self.nomorTeleponLineEdit.setText("" + str(tp[2]))
                    self.jarakTempuhKmLineEdit.setText("" + str(tp[3]))
                    self.potonganAutoLineEdit.setText("" + str(tp[4]))
                    self.hargaAutoLineEdit.setText("" + str(tp[5]))
            else:
                self.messagebox("INFO", "Data tidak ditemukan")

        def clear(self):
            self.namaCostumerLineEdit.clear()
            self.idCostumerAutoLineEdit.clear()
            self.nomorTeleponLineEdit.clear()
            self.jarakTempuhKmLineEdit.clear()
            self.potonganAutoLineEdit.clear()
            self.hargaAutoLineEdit.clear()

        def cari(self):
            nama = self.namaCostumerLineEdit.text()
            nope = self.nomorTeleponLineEdit.text()
            db = pymysql.connect(db='db_pyojol', user='root', passwd='',
                                 host='localhost', port=3306, autocommit=True)
            cursor = db.cursor()
            cursor.execute(
                "SELECT nama, id_customer, nope, jtempuh, potongan, harga FROM customer WHERE nama='"+str(nama)+"' OR nope='"+str(nope)+"'")
            query = cursor.fetchall()
            self.tableWidget.setRowCount(0)
            for row_number, row_data in enumerate(query):
                self.tableWidget.insertRow(row_number)
                for column_number, data in enumerate(row_data):
                    self.tableWidget.setItem(
                        row_number, column_number, QtWidgets.QTableWidgetItem(str(data)))

        def setupUi(self, MainWindow):
            self.koneksi()
            MainWindow.setObjectName("MainWindow")
            MainWindow.resize(543, 521)
            self.centralwidget = QtWidgets.QWidget(MainWindow)
            self.centralwidget.setObjectName("centralwidget")
            self.formLayoutWidget = QtWidgets.QWidget(self.centralwidget)
            self.formLayoutWidget.setGeometry(QtCore.QRect(10, 10, 521, 181))
            self.formLayoutWidget.setObjectName("formLayoutWidget")
            self.formLayout_2 = QtWidgets.QFormLayout(self.formLayoutWidget)
            self.formLayout_2.setContentsMargins(0, 0, 0, 0)
            self.formLayout_2.setObjectName("formLayout_2")
            self.namaCostumerLabel = QtWidgets.QLabel(self.formLayoutWidget)
            self.namaCostumerLabel.setObjectName("namaCostumerLabel")
            self.formLayout_2.setWidget(
                0, QtWidgets.QFormLayout.LabelRole, self.namaCostumerLabel)
            self.namaCostumerLineEdit = QtWidgets.QLineEdit(self.formLayoutWidget)
            self.namaCostumerLineEdit.setObjectName("namaCostumerLineEdit")
            self.formLayout_2.setWidget(
                0, QtWidgets.QFormLayout.FieldRole, self.namaCostumerLineEdit)
            self.idCostumerAutoLabel = QtWidgets.QLabel(self.formLayoutWidget)
            font = QtGui.QFont()
            font.setPointSize(12)
            self.idCostumerAutoLabel.setFont(font)
            self.idCostumerAutoLabel.setObjectName("idCostumerAutoLabel")
            self.formLayout_2.setWidget(
                1, QtWidgets.QFormLayout.LabelRole, self.idCostumerAutoLabel)
            self.idCostumerAutoLineEdit = QtWidgets.QLineEdit(
                self.formLayoutWidget)
            self.idCostumerAutoLineEdit.setEnabled(False)
            self.idCostumerAutoLineEdit.setObjectName("idCostumerAutoLineEdit")
            self.formLayout_2.setWidget(
                1, QtWidgets.QFormLayout.FieldRole, self.idCostumerAutoLineEdit)
            self.nomorTeleponLabel = QtWidgets.QLabel(self.formLayoutWidget)
            font = QtGui.QFont()
            font.setPointSize(12)
            self.nomorTeleponLabel.setFont(font)
            self.nomorTeleponLabel.setObjectName("nomorTeleponLabel")
            self.formLayout_2.setWidget(
                2, QtWidgets.QFormLayout.LabelRole, self.nomorTeleponLabel)
            self.nomorTeleponLineEdit = QtWidgets.QLineEdit(self.formLayoutWidget)
            self.nomorTeleponLineEdit.setObjectName("nomorTeleponLineEdit")
            self.formLayout_2.setWidget(
                2, QtWidgets.QFormLayout.FieldRole, self.nomorTeleponLineEdit)
            self.jarakTempuhKmLabel = QtWidgets.QLabel(self.formLayoutWidget)
            font = QtGui.QFont()
            font.setPointSize(12)
            self.jarakTempuhKmLabel.setFont(font)
            self.jarakTempuhKmLabel.setObjectName("jarakTempuhKmLabel")
            self.formLayout_2.setWidget(
                3, QtWidgets.QFormLayout.LabelRole, self.jarakTempuhKmLabel)
            self.jarakTempuhKmLineEdit = QtWidgets.QLineEdit(self.formLayoutWidget)
            self.jarakTempuhKmLineEdit.setObjectName("jarakTempuhKmLineEdit")
            self.formLayout_2.setWidget(
                3, QtWidgets.QFormLayout.FieldRole, self.jarakTempuhKmLineEdit)
            self.potonganAutoLabel = QtWidgets.QLabel(self.formLayoutWidget)
            font = QtGui.QFont()
            font.setPointSize(12)
            self.potonganAutoLabel.setFont(font)
            self.potonganAutoLabel.setObjectName("potonganAutoLabel")
            self.formLayout_2.setWidget(
                4, QtWidgets.QFormLayout.LabelRole, self.potonganAutoLabel)
            self.potonganAutoLineEdit = QtWidgets.QLineEdit(self.formLayoutWidget)
            self.potonganAutoLineEdit.setEnabled(False)
            self.potonganAutoLineEdit.setObjectName("potonganAutoLineEdit")
            self.formLayout_2.setWidget(
                4, QtWidgets.QFormLayout.FieldRole, self.potonganAutoLineEdit)
            self.hargaAutoLabel = QtWidgets.QLabel(self.formLayoutWidget)
            font = QtGui.QFont()
            font.setPointSize(12)
            self.hargaAutoLabel.setFont(font)
            self.hargaAutoLabel.setObjectName("hargaAutoLabel")
            self.formLayout_2.setWidget(
                5, QtWidgets.QFormLayout.LabelRole, self.hargaAutoLabel)
            self.hargaAutoLineEdit = QtWidgets.QLineEdit(self.formLayoutWidget)
            self.hargaAutoLineEdit.setEnabled(False)
            self.hargaAutoLineEdit.setObjectName("hargaAutoLineEdit")
            self.formLayout_2.setWidget(
                5, QtWidgets.QFormLayout.FieldRole, self.hargaAutoLineEdit)
            self.orderButton = QtWidgets.QPushButton(self.centralwidget)
            self.orderButton.setGeometry(QtCore.QRect(10, 430, 121, 41))
            self.orderButton.setObjectName("orderButton")
            self.orderButton.clicked.connect(self.save)
            self.orderButton.clicked.connect(self.tampil)
            self.orderButton.clicked.connect(self.clear)
            self.editButton = QtWidgets.QPushButton(self.centralwidget)
            self.editButton.setGeometry(QtCore.QRect(140, 430, 121, 41))
            self.editButton.setObjectName("editButton")
            self.editButton.clicked.connect(self.update)
            self.editButton.clicked.connect(self.tampil)
            self.editButton.clicked.connect(self.clear)
            self.editButton_2 = QtWidgets.QPushButton(self.centralwidget)
            self.editButton_2.setGeometry(QtCore.QRect(280, 430, 121, 41))
            self.editButton_2.setObjectName("editButton_2")
            self.editButton_2.clicked.connect(self.cari)
            self.editButton_2.clicked.connect(self.form)
            self.deleteButton = QtWidgets.QPushButton(self.centralwidget)
            self.deleteButton.setGeometry(QtCore.QRect(410, 430, 121, 41))
            self.deleteButton.setObjectName("deleteButton")
            self.deleteButton.clicked.connect(self.delete)
            self.tableWidget = QtWidgets.QTableWidget(self.centralwidget)
            self.tableWidget.setGeometry(QtCore.QRect(10, 210, 521, 201))
            self.tableWidget.setRowCount(5)
            self.tableWidget.setColumnCount(6)
            self.tableWidget.setObjectName("tableWidget")
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(0, item)
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(1, item)
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(2, item)
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(3, item)
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(4, item)
            item = QtWidgets.QTableWidgetItem()
            self.tableWidget.setHorizontalHeaderItem(5, item)
            self.reloadButton = QtWidgets.QPushButton(self.centralwidget)
            self.reloadButton.setGeometry(QtCore.QRect(430, 185, 101, 21))
            self.reloadButton.setObjectName("reloadButton")
            self.reloadButton.clicked.connect(self.tampil)
            self.reloadButton.clicked.connect(self.clear)
            MainWindow.setCentralWidget(self.centralwidget)
            self.menubar = QtWidgets.QMenuBar(MainWindow)
            self.menubar.setGeometry(QtCore.QRect(0, 0, 543, 21))
            self.menubar.setObjectName("menubar")
            MainWindow.setMenuBar(self.menubar)
            self.statusbar = QtWidgets.QStatusBar(MainWindow)
            self.statusbar.setObjectName("statusbar")
            MainWindow.setStatusBar(self.statusbar)

            self.retranslateUi(MainWindow)
            QtCore.QMetaObject.connectSlotsByName(MainWindow)

        def retranslateUi(self, MainWindow):
            _translate = QtCore.QCoreApplication.translate
            MainWindow.setWindowTitle(_translate("MainWindow", "MainWindow"))
            self.namaCostumerLabel.setText(_translate(
                "MainWindow", "<html><head/><body><p><span style=\" font-size:12pt;\">Nama Costumer</span></p></body></html>"))
            self.idCostumerAutoLabel.setText(
                _translate("MainWindow", "Id Costumer (Auto)"))
            self.nomorTeleponLabel.setText(
                _translate("MainWindow", "Nomor Telepon"))
            self.jarakTempuhKmLabel.setText(
                _translate("MainWindow", "Jarak Tempuh (km)"))
            self.potonganAutoLabel.setText(
                _translate("MainWindow", "Potongan (Auto)"))
            self.hargaAutoLabel.setText(_translate("MainWindow", "Harga (Auto)"))
            self.orderButton.setText(_translate("MainWindow", "Order"))
            self.editButton.setText(_translate("MainWindow", "Edit"))
            self.editButton_2.setText(_translate("MainWindow", "Search Data"))
            self.deleteButton.setText(_translate("MainWindow", "Delete"))
            item = self.tableWidget.horizontalHeaderItem(0)
            item.setText(_translate("MainWindow", "Nama Customer"))
            item = self.tableWidget.horizontalHeaderItem(1)
            item.setText(_translate("MainWindow", "Id Costumer"))
            item = self.tableWidget.horizontalHeaderItem(2)
            item.setText(_translate("MainWindow", "Nomor Telepon"))
            item = self.tableWidget.horizontalHeaderItem(3)
            item.setText(_translate("MainWindow", "Jarak Tempuh"))
            item = self.tableWidget.horizontalHeaderItem(4)
            item.setText(_translate("MainWindow", "Potongan"))
            item = self.tableWidget.horizontalHeaderItem(5)
            item.setText(_translate("MainWindow", "Harga"))
            self.reloadButton.setText(_translate("MainWindow", "Reload"))


    if __name__ == "__main__":
        import sys
        app = QtWidgets.QApplication(sys.argv)
        MainWindow = QtWidgets.QMainWindow()
        ui = Ui_MainWindow()
        ui.setupUi(MainWindow)
        MainWindow.show()
        sys.exit(app.exec_())
