//////
using System;

public class Student
{
    public string StudentId { get; set; }
    public string FullName { get; set; }
    public int Age { get; set; }
    public string Gender { get; set; }
    public string AvatarPath { get; set; }

    public Student() { }

    public Student(string id, string name, int age, string gender, string avatar)
    {
        StudentId = id;
        FullName = name;
        Age = age;
        Gender = gender;
        AvatarPath = avatar;
    }
}
////////
using System.ComponentModel;
using System.Text;

namespace bai6_13_10
{
    public partial class Form1 : Form
    {
        private BindingList<Student> studentList;
        private BindingSource bindingSource;
        public Form1()
        {
            InitializeComponent();
        }
        private void Form1_Load(object sender, EventArgs e)
        {
            // 1. Khởi tạo dữ liệu mẫu tối giản
            studentList = new BindingList<Student>()
            {
                new Student("SV01", "Nguyễn Văn A", 20, "Nam", ""),
                new Student("SV02", "Trần Thị B", 21, "Nữ", "")
            };

            bindingSource = new BindingSource();
            bindingSource.DataSource = studentList;


            dgvStudents.DataSource = bindingSource;


            txtStudentId.DataBindings.Add("Text", bindingSource, "StudentId", true, DataSourceUpdateMode.OnPropertyChanged);
            txtFullName.DataBindings.Add("Text", bindingSource, "FullName", true, DataSourceUpdateMode.OnPropertyChanged);
            txtAge.DataBindings.Add("Text", bindingSource, "Age", true, DataSourceUpdateMode.OnPropertyChanged);
            cboGender.DataBindings.Add("SelectedItem", bindingSource, "Gender", true, DataSourceUpdateMode.OnPropertyChanged);


            bindingSource.CurrentChanged += BindingSource_CurrentChanged;


            UpdateAvatarDisplay();
        }

        private void BindingSource_CurrentChanged(object sender, EventArgs e)
        {
            UpdateAvatarDisplay();
        }


        private void UpdateAvatarDisplay()
        {
            Student currentStudent = bindingSource.Current as Student;
            if (currentStudent != null && !string.IsNullOrEmpty(currentStudent.AvatarPath) && File.Exists(currentStudent.AvatarPath))
            {
                picAvatar.ImageLocation = currentStudent.AvatarPath;
            }
            else
            {
                picAvatar.Image = null; // Xóa ảnh cũ nếu sinh viên không có đường dẫn hoặc file lỗi
            }
        }


        private void btnChooseAvatar_Click(object sender, EventArgs e)
        {
            Student currentStudent = bindingSource.Current as Student;
            if (currentStudent == null) return;

            using (OpenFileDialog ofd = new OpenFileDialog())
            {
                ofd.Filter = "Image Files(*.jpg; *.png; *.bmp)|*.jpg; *.png; *.bmp";
                if (ofd.ShowDialog() == DialogResult.OK)
                {
                    currentStudent.AvatarPath = ofd.FileName;
                    UpdateAvatarDisplay();
                }
            }
        }


        private void btnAdd_Click(object sender, EventArgs e)
        {

            Student newStudent = new Student("", "", 18, "Nam", "");
            studentList.Add(newStudent);
            bindingSource.MoveLast(); // Di chuyển con trỏ tới dòng mới thêm
        }


        private void btnDelete_Click(object sender, EventArgs e)
        {
            if (bindingSource.Current != null)
            {
                bindingSource.RemoveCurrent();
            }
        }


        private void btnUpdate_Click(object sender, EventArgs e)
        {
            bindingSource.ResetCurrentItem();
            MessageBox.Show("Cập nhật dữ liệu thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }


        private void btnExportCsv_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog sfd = new SaveFileDialog())
            {
                sfd.Filter = "CSV files (*.csv)|*.csv";
                sfd.FileName = "DanhSachSinhVien.csv";

                if (sfd.ShowDialog() == DialogResult.OK)
                {
                    try
                    {
                        StringBuilder csvContent = new StringBuilder();

                        csvContent.AppendLine("Mã SV,Họ tên,Tuổi,Giới tính,Đường dẫn ảnh");


                        foreach (Student std in studentList)
                        {
                            csvContent.AppendLine($"{std.StudentId},{std.FullName},{std.Age},{std.Gender},{std.AvatarPath}");
                        }


                        File.WriteAllText(sfd.FileName, csvContent.ToString(), Encoding.UTF8);
                        MessageBox.Show("Xuất file CSV thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show($"Lỗi khi lưu file: {ex.Message}", "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }
    }
}
