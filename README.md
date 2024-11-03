
# Aplikasi Pengelolaan Kontak

Latihan 3 Modul Aplikasi Pengelolaan Kontak

# Deskripsi
Aplikasi Pengelolaan Kontak ini memungkinkan pengguna untuk menyimpan dan mengelola informasi kontak, seperti:
- Nama
- Nomor Telepon
- Kategori Kontak
Aplikasi ini menggunakan database SQLite untuk menyimpan data kontak dan mendukung fitur untuk menambah, mengedit, serta menghapus kontak. Pengguna juga dapat mengelompokkan kontak berdasarkan kategori (keluarga, teman, atau kerja) menggunakan JComboBox.
# Features

Fitur tambahan dari aplikasi ini meliputi:

1. Pencarian Kontak: Pengguna dapat mencari kontak berdasarkan nama atau nomor telepon, dan hasilnya akan ditampilkan di JTable.
2. Validasi Input: Terdapat validasi input untuk memastikan nomor telepon hanya terdiri dari angka dan memiliki panjang yang sesuai.
3. Ekspor/Impor Kontak: Aplikasi mendukung fitur untuk mengekspor daftar kontak ke dalam file CSV dan mengimpor kontak dari file CSV ke dalam database.

# Komponen GUI

Aplikasi ini menggunakan beberapa komponen GUI utama, di antaranya:

- JFrame
- JPanel
- JLabel
- JTextField
- JButton
- JList
- JComboBox
- JTable
- JScrollPane

# Event
Event handling yang digunakan dalam aplikasi ini meliputi:

ActionListener untuk menangani aksi pada tombol Tambah, Edit, Hapus, dan Cari Kontak.
ItemListener untuk menangani perubahan pilihan kategori kontak pada JComboBox.

# Logika Program

Program ini menggunakan database SQLite sebagai tempat penyimpanan data kontak dan mendukung operasi CRUD (Create, Read, Update, Delete).
# Dokumentasi Code

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import javax.swing.JOptionPane;
import javax.swing.JTable;
import javax.swing.table.DefaultTableModel;
import java.io.FileWriter;
import java.io.IOException;
import java.io.BufferedReader;
import java.io.FileReader;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;




public class AplikasiPencariKontak extends javax.swing.JFrame {
    private DefaultTableModel tableModel;
    public AplikasiPencariKontak() {
    initComponents();
    // Inisialisasi model tabel
    tableModel = new DefaultTableModel(new String[]{"ID", "Nama", "Telepon", "Kategori"}, 0);
    // Menghubungkan model ke JTable
    kontakTable.setModel(tableModel);
}
    
    
    
public class DatabaseHelper {
    public Connection connect() {
        Connection conn = null;
        try{
            Class.forName("org.sqlite.JDBC");
            Connection con = DriverManager.getConnection("jdbc:sqlite:kontak.sqlite");
            System.out.println("Connect Berhasil");
                     
            return con;
           }catch (Exception e){
            System.out.println("Connect Gagal"+e);
            return null;
           }
    }
}



    
    public void tambahKontak(String nama, String telepon, String kategori) {
    DatabaseHelper dbHelper = new DatabaseHelper();
    try (Connection conn = dbHelper.connect()) {
        String sql = "INSERT INTO kontak (nama, telepon, kategori) VALUES (?, ?, ?)";
        PreparedStatement pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, nama);
        pstmt.setString(2, telepon);
        pstmt.setString(3, kategori);
        pstmt.executeUpdate();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
    
    public List<String[]> getKontakList() {
    List<String[]> kontakList = new ArrayList<>();
    DatabaseHelper dbHelper = new DatabaseHelper();
    try (Connection conn = dbHelper.connect()) {
        String sql = "SELECT * FROM kontak";
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery(sql);

        while (rs.next()) {
            String[] kontak = new String[4];
            kontak[0] = String.valueOf(rs.getInt("id"));
            kontak[1] = rs.getString("nama");
            kontak[2] = rs.getString("telepon");
            kontak[3] = rs.getString("kategori");
            kontakList.add(kontak);
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    return kontakList;
}
    
    public void updateKontak(int id, String nama, String telepon, String kategori) {
    DatabaseHelper dbHelper = new DatabaseHelper();
    try (Connection conn = dbHelper.connect()) {
        String sql = "UPDATE kontak SET nama = ?, telepon = ?, kategori = ? WHERE id = ?";
        PreparedStatement pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, nama);
        pstmt.setString(2, telepon);
        pstmt.setString(3, kategori);
        pstmt.setInt(4, id);
        pstmt.executeUpdate();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
    
    public void hapusKontak(int id) {
    DatabaseHelper dbHelper = new DatabaseHelper();
    try (Connection conn = dbHelper.connect()) {
        String sql = "DELETE FROM kontak WHERE id = ?";
        PreparedStatement pstmt = conn.prepareStatement(sql);
        pstmt.setInt(1, id);
        pstmt.executeUpdate();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
    
   public void tampilkanKontak(DefaultTableModel model) {
    model.setRowCount(0); // Menghapus baris lama
    List<String[]> kontakList = getKontakList(); // Mendapatkan daftar kontak dari database

    for (String[] kontak : kontakList) {
        model.addRow(kontak); // Tambahkan kontak ke dalam JTable
    }
}
   
public List<String[]> cariKontak(String keyword) {
    List<String[]> kontakList = new ArrayList<>();
    DatabaseHelper dbHelper = new DatabaseHelper();
    
    try (Connection conn = dbHelper.connect()) {
        String sql = "SELECT * FROM kontak WHERE nama LIKE ? OR telepon LIKE ?";
        PreparedStatement pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, "%" + keyword + "%");
        pstmt.setString(2, "%" + keyword + "%");
        
        ResultSet rs = pstmt.executeQuery();
        
        while (rs.next()) {
            String[] kontak = new String[4];
            kontak[0] = String.valueOf(rs.getInt("id"));
            kontak[1] = rs.getString("nama");
            kontak[2] = rs.getString("telepon");
            kontak[3] = rs.getString("kategori");
            kontakList.add(kontak);
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    
    return kontakList;
}

   
   public void tampilkanKontakBerdasarkanKategori(String kategori) {
    tableModel.setRowCount(0); // Menghapus semua baris lama di tabel
    if (kategori == null || tableModel == null) {
        System.out.println("Kategori atau tableModel tidak boleh null");
        return;
    }

    List<String[]> kontakList = getKontakList(); // Dapatkan daftar semua kontak

    // Filter kontak berdasarkan kategori
    for (String[] kontak : kontakList) {
        if (kontak[3].equals(kategori)) { // Asumsi kontak[3] adalah kolom kategori
            tableModel.addRow(kontak); // Tambahkan kontak yang sesuai ke tabel
        }
    }
}

   
   public boolean validasiNomorTelepon(String telepon) {
    // Cek apakah panjang nomor telepon sesuai
    if (telepon.length() < 10 || telepon.length() > 13) {
        return false;
    }
    // Cek apakah hanya berisi angka
    for (char c : telepon.toCharArray()) {
        if (!Character.isDigit(c)) {
            return false;
        }
    }
    return true;
}

   
public void eksporKeCSV(String filePath) {
    try (FileWriter writer = new FileWriter(filePath)) {
        // Tulis header kolom
        writer.write("ID,Nama,Telepon,Kategori\n");
        
        // Iterasi melalui data di tableModel
        for (int i = 0; i < tableModel.getRowCount(); i++) {
            String id = tableModel.getValueAt(i, 0).toString();
            String nama = tableModel.getValueAt(i, 1).toString();
            String telepon = tableModel.getValueAt(i, 2).toString();
            String kategori = tableModel.getValueAt(i, 3).toString();
            
            writer.write(id + "," + nama + "," + telepon + "," + kategori + "\n");
        }
        
        writer.flush();
        JOptionPane.showMessageDialog(this, "Data berhasil diekspor ke " + filePath, "Ekspor Berhasil", JOptionPane.INFORMATION_MESSAGE);
    } catch (IOException e) {
        e.printStackTrace();
        JOptionPane.showMessageDialog(this, "Terjadi kesalahan saat mengekspor data", "Error", JOptionPane.ERROR_MESSAGE);
    }
}


public void imporDariCSV(String filePath) {
    DatabaseHelper dbHelper = new DatabaseHelper();
    
    try (BufferedReader reader = new BufferedReader(new FileReader(filePath));
         Connection conn = dbHelper.connect()) {
         
        String line;
        
        // Lewati header
        reader.readLine();
        
        // Iterasi setiap baris
        while ((line = reader.readLine()) != null) {
            String[] data = line.split(",");
            
            if (data.length == 4) { // Pastikan ada 4 kolom data
                String id = data[0];
                String nama = data[1];
                String telepon = data[2];
                String kategori = data[3];
                
                // Tambahkan data ke database
                String sql = "INSERT INTO kontak (id, nama, telepon, kategori) VALUES (?, ?, ?, ?)";
                PreparedStatement pstmt = conn.prepareStatement(sql);
                pstmt.setInt(1, Integer.parseInt(id));
                pstmt.setString(2, nama);
                pstmt.setString(3, telepon);
                pstmt.setString(4, kategori);
                pstmt.executeUpdate();
            }
        }
        
        JOptionPane.showMessageDialog(this, "Data berhasil diimpor dari " + filePath, "Impor Berhasil", JOptionPane.INFORMATION_MESSAGE);
        
    } catch (IOException | SQLException e) {
        e.printStackTrace();
        JOptionPane.showMessageDialog(this, "Terjadi kesalahan saat mengimpor data", "Error", JOptionPane.ERROR_MESSAGE);
    }
}





    
 





    
    

    /**
     * This method is called from within the constructor to initialize the form.
     * WARNING: Do NOT modify this code. The content of this method is always
     * regenerated by the Form Editor.
     */
    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">                          
    private void initComponents() {

        jPanel1 = new javax.swing.JPanel();
        jLabel2 = new javax.swing.JLabel();
        jLabel3 = new javax.swing.JLabel();
        namaTextField = new javax.swing.JTextField();
        teleponTextField = new javax.swing.JTextField();
        jLabel4 = new javax.swing.JLabel();
        kategoriComboBox = new javax.swing.JComboBox<>();
        tambahButton = new javax.swing.JButton();
        editButton = new javax.swing.JButton();
        hapusButton = new javax.swing.JButton();
        cariButton = new javax.swing.JButton();
        jScrollPane1 = new javax.swing.JScrollPane();
        kontakTable = new javax.swing.JTable();
        eksporButton = new javax.swing.JButton();
        imporButton = new javax.swing.JButton();

        setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);

        jPanel1.setBorder(javax.swing.BorderFactory.createTitledBorder(null, "Aplikasi Pengelola Kontak", javax.swing.border.TitledBorder.DEFAULT_JUSTIFICATION, javax.swing.border.TitledBorder.DEFAULT_POSITION, new java.awt.Font("Times New Roman", 3, 18))); // NOI18N

        jLabel2.setText("Nama");

        jLabel3.setText("No Telepon");

        jLabel4.setText("Kategori");

        kategoriComboBox.setModel(new javax.swing.DefaultComboBoxModel<>(new String[] { "Pilih Kategori", "Keluarga", "Teman", "Kerja", " " }));
        kategoriComboBox.addItemListener(new java.awt.event.ItemListener() {
            public void itemStateChanged(java.awt.event.ItemEvent evt) {
                kategoriComboBoxItemStateChanged(evt);
            }
        });

        tambahButton.setText("Tambah");
        tambahButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                tambahButtonActionPerformed(evt);
            }
        });

        editButton.setText("Ubah");
        editButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                editButtonActionPerformed(evt);
            }
        });

        hapusButton.setText("Hapus");
        hapusButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                hapusButtonActionPerformed(evt);
            }
        });

        cariButton.setText("Cari");
        cariButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                cariButtonActionPerformed(evt);
            }
        });

        kontakTable.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {
                {null, null, null},
                {null, null, null},
                {null, null, null},
                {null, null, null}
            },
            new String [] {
                "Nama", "No Telepon", "Kategori"
            }
        ));
        jScrollPane1.setViewportView(kontakTable);

        eksporButton.setText("Ekspor Data");
        eksporButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                eksporButtonActionPerformed(evt);
            }
        });

        imporButton.setText("Impor Data");
        imporButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                imporButtonActionPerformed(evt);
            }
        });

        javax.swing.GroupLayout jPanel1Layout = new javax.swing.GroupLayout(jPanel1);
        jPanel1.setLayout(jPanel1Layout);
        jPanel1Layout.setHorizontalGroup(
            jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(jPanel1Layout.createSequentialGroup()
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(jPanel1Layout.createSequentialGroup()
                        .addContainerGap()
                        .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addGroup(jPanel1Layout.createSequentialGroup()
                                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                    .addComponent(jLabel2)
                                    .addComponent(jLabel3)
                                    .addComponent(jLabel4))
                                .addGap(117, 117, 117)
                                .addComponent(kategoriComboBox, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))))
                    .addGroup(jPanel1Layout.createSequentialGroup()
                        .addContainerGap()
                        .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addComponent(namaTextField, javax.swing.GroupLayout.Alignment.TRAILING, javax.swing.GroupLayout.PREFERRED_SIZE, 267, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addGroup(jPanel1Layout.createSequentialGroup()
                                .addComponent(tambahButton)
                                .addGap(36, 36, 36)
                                .addComponent(editButton, javax.swing.GroupLayout.PREFERRED_SIZE, 80, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addGap(30, 30, 30)
                                .addComponent(hapusButton, javax.swing.GroupLayout.PREFERRED_SIZE, 84, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addGap(31, 31, 31)
                                .addComponent(cariButton, javax.swing.GroupLayout.PREFERRED_SIZE, 79, javax.swing.GroupLayout.PREFERRED_SIZE))
                            .addComponent(teleponTextField, javax.swing.GroupLayout.Alignment.TRAILING, javax.swing.GroupLayout.PREFERRED_SIZE, 267, javax.swing.GroupLayout.PREFERRED_SIZE)))
                    .addGroup(jPanel1Layout.createSequentialGroup()
                        .addGap(44, 44, 44)
                        .addComponent(eksporButton)
                        .addGap(140, 140, 140)
                        .addComponent(imporButton)))
                .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
        );
        jPanel1Layout.setVerticalGroup(
            jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(jPanel1Layout.createSequentialGroup()
                .addGap(65, 65, 65)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel2)
                    .addComponent(namaTextField, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addGap(18, 18, 18)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel3)
                    .addComponent(teleponTextField, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addGap(18, 18, 18)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel4)
                    .addComponent(kategoriComboBox, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addGap(33, 33, 33)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(tambahButton)
                    .addComponent(editButton)
                    .addComponent(hapusButton)
                    .addComponent(cariButton))
                .addGap(18, 18, 18)
                .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 233, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(18, 18, 18)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(eksporButton)
                    .addComponent(imporButton))
                .addContainerGap(14, Short.MAX_VALUE))
        );

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(getContentPane());
        getContentPane().setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addContainerGap()
                .addComponent(jPanel1, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                .addContainerGap())
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addContainerGap()
                .addComponent(jPanel1, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                .addContainerGap())
        );

        pack();
    }// </editor-fold>                        

    private void tambahButtonActionPerformed(java.awt.event.ActionEvent evt) {                                             
      tambahButton.addActionListener(e -> {
    String nama = namaTextField.getText();
    String telepon = teleponTextField.getText();
    String kategori = (String) kategoriComboBox.getSelectedItem();
    
    // Validasi nomor telepon
    if (!validasiNomorTelepon(telepon)) {
        JOptionPane.showMessageDialog(this, "Nomor telepon harus berisi angka dan memiliki panjang antara 10-13 digit.", "Error", JOptionPane.ERROR_MESSAGE);
        return; // Batalkan proses jika validasi gagal
    }

    // Jika validasi berhasil, lanjutkan dengan menambah kontak
    tambahKontak(nama, telepon, kategori);
    tampilkanKontak(tableModel); // Refresh JTable setelah tambah data
});



    }                                            

    private void editButtonActionPerformed(java.awt.event.ActionEvent evt) {                                           
        editButton.addActionListener(e -> {
    int selectedRow = kontakTable.getSelectedRow();
    if (selectedRow != -1) { // Pastikan ada baris yang dipilih
        int id = Integer.parseInt((String) tableModel.getValueAt(selectedRow, 0));
        String nama = namaTextField.getText();
        String telepon = teleponTextField.getText();
        String kategori = (String) kategoriComboBox.getSelectedItem();
        updateKontak(id, nama, telepon, kategori);
        tampilkanKontak(tableModel); // Refresh JTable setelah update data
    }
});

    }                                          

    private void hapusButtonActionPerformed(java.awt.event.ActionEvent evt) {                                            
        hapusButton.addActionListener(e -> {
    int selectedRow = kontakTable.getSelectedRow();
    if (selectedRow != -1) { // Pastikan ada baris yang dipilih
        int id = Integer.parseInt((String) tableModel.getValueAt(selectedRow, 0));
        hapusKontak(id);
        tampilkanKontak(tableModel); // Refresh JTable setelah hapus data
    }
});

    }                                           

    private void cariButtonActionPerformed(java.awt.event.ActionEvent evt) {                                           
        cariButton.addActionListener(e -> {
    String keyword = namaTextField.getText();
    List<String[]> hasilCari = cariKontak(keyword); // cariKontak adalah metode pencarian
    tableModel.setRowCount(0); // Bersihkan tabel sebelum menampilkan hasil
    for (String[] kontak : hasilCari) {
        tableModel.addRow(kontak);
    }
});

    }                                          

    private void kategoriComboBoxItemStateChanged(java.awt.event.ItemEvent evt) {                                                  
        kategoriComboBox.addItemListener(e -> {
    if (e.getStateChange() == java.awt.event.ItemEvent.SELECTED) {
        String kategoriTerpilih = (String) e.getItem();
        // Panggil metode untuk menampilkan kontak berdasarkan kategori
        tampilkanKontakBerdasarkanKategori(kategoriTerpilih);
    }
});

    }                                                 

    private void eksporButtonActionPerformed(java.awt.event.ActionEvent evt) {                                             
        eksporButton.addActionListener(e -> {
    String filePath = "kontak.csv"; // Tentukan path atau gunakan dialog untuk memilih path
    eksporKeCSV(filePath);
});
    }                                            

    private void imporButtonActionPerformed(java.awt.event.ActionEvent evt) {                                            
        imporButton.addActionListener(e -> {
    String filePath = "kontak.csv"; // Tentukan path atau gunakan dialog untuk memilih path
    imporDariCSV(filePath);
    tampilkanKontak(tableModel); // Refresh JTable setelah impor data
});
    }                                           

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        /* Set the Nimbus look and feel */
        //<editor-fold defaultstate="collapsed" desc=" Look and feel setting code (optional) ">
        /* If Nimbus (introduced in Java SE 6) is not available, stay with the default look and feel.
         * For details see http://download.oracle.com/javase/tutorial/uiswing/lookandfeel/plaf.html 
         */
        try {
            for (javax.swing.UIManager.LookAndFeelInfo info : javax.swing.UIManager.getInstalledLookAndFeels()) {
                if ("Nimbus".equals(info.getName())) {
                    javax.swing.UIManager.setLookAndFeel(info.getClassName());
                    break;
                }
            }
        } catch (ClassNotFoundException ex) {
            java.util.logging.Logger.getLogger(AplikasiPencariKontak.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (InstantiationException ex) {
            java.util.logging.Logger.getLogger(AplikasiPencariKontak.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (IllegalAccessException ex) {
            java.util.logging.Logger.getLogger(AplikasiPencariKontak.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (javax.swing.UnsupportedLookAndFeelException ex) {
            java.util.logging.Logger.getLogger(AplikasiPencariKontak.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        }
        //</editor-fold>

        /* Create and display the form */
        java.awt.EventQueue.invokeLater(new Runnable() {
            public void run() {
                new AplikasiPencariKontak().setVisible(true);
            }
        });
    }

    // Variables declaration - do not modify                     
    private javax.swing.JButton cariButton;
    private javax.swing.JButton editButton;
    private javax.swing.JButton eksporButton;
    private javax.swing.JButton hapusButton;
    private javax.swing.JButton imporButton;
    private javax.swing.JLabel jLabel2;
    private javax.swing.JLabel jLabel3;
    private javax.swing.JLabel jLabel4;
    private javax.swing.JPanel jPanel1;
    private javax.swing.JScrollPane jScrollPane1;
    private javax.swing.JComboBox<String> kategoriComboBox;
    private javax.swing.JTable kontakTable;
    private javax.swing.JTextField namaTextField;
    private javax.swing.JButton tambahButton;
    private javax.swing.JTextField teleponTextField;
    // End of variables declaration                   
}








```


## Authors

Muhammad Irwan Firmanto
2210010582 5B Reg Pagi Banjarmasin

- [@eternalsugarzy](https://www.github.com/eternalsugarzy)


## Screenshots



