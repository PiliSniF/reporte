# reporte
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Reflection;
using System.IO;
using iTextSharp;
using iTextSharp.text.pdf;
using iTextSharp.text;
using iTextSharp.text.pdf.parser;
using System.util.collections;
using System.Net.Mail;
using MySql.Data.MySqlClient;
using MySql.Data.Types;
using MySql.Data.Common;

namespace snif
{
    public partial class ReporteAula : Form
    {

        public static class Global
        {
            public static string fecha = "";
           
        }
        
        public ReporteAula()
        {
            InitializeComponent();
            rellenar_combo();
        }


        public MySqlDataReader sede() {

            MySqlDataReader sede;

            Conexion con = new Conexion();
            string nombreProfes = " SELECT " +
                                    " s.id AS id_sede, " +
                                    " s.name AS nombre_sede " +
                                    " FROM " +
                                    "   mdl_course_categories s " +
                                    " WHERE " +
                                    "   s.id BETWEEN 2 AND 4";

            con.comando = new MySqlCommand(nombreProfes, con.conex);
            sede = con.comando.ExecuteReader();

            return sede;
        }

        public MySqlDataReader carrera()
        {

            MySqlDataReader carrera;
            Conexion con = new Conexion();
            string nombreProfes = " SELECT " +
                                    " s.id AS id_sede, " +
                                    " s.name AS nombre_sede " +
                                    " FROM " +
                                    "   mdl_course_categories s " +
                                    " WHERE " +
                                    "   s.id BETWEEN 2 AND 4";

            con.comando = new MySqlCommand(nombreProfes, con.conex);
            carrera = con.comando.ExecuteReader();
            return carrera;
        }
        public void rellenar_combo() {
            MySqlDataReader retorno = sede();
            ComboboxItem item = new ComboboxItem();
            item.Value = "-1";
            item.Text = "Ninguna Sede";

            cmb_sede.Items.Add(item);

            while (retorno.Read())
            {


                ComboboxItem aux = new ComboboxItem();
                aux.Value = retorno["id_sede"].ToString();
                aux.Text = retorno["nombre_sede"].ToString();


                cmb_sede.Items.Add(aux);

            }


            cmb_sede.SelectedIndex = 0;


            retorno = carrera();
            ComboboxItem item2 = new ComboboxItem();
            item2.Value = "-1";
            item2.Text = "Ninguna Carrera";

            cmb_carrera.Items.Add(item2);

            while (retorno.Read())
            {


                ComboboxItem aux = new ComboboxItem();
                aux.Value = retorno["id_sede"].ToString();
                aux.Text = retorno["nombre_sede"].ToString();


                cmb_carrera.Items.Add(aux);

            }


            cmb_sede.SelectedIndex = 0;

        
        }

        public class ComboboxItem
        {
            public string Text { get; set; }
            public string Value { get; set; }

            public override string ToString()
            {
                return Text;
            }

        }

        public void rellenar() {

            Conexion con = new Conexion();
            MySqlDataReader profes;
            string nombreProfes ="SELECT "+
                                    " p.name AS carrera," +
                                    " c.id AS idCurso," +
                                    " c.fullname AS asignatura," +
                                    " u.firstname AS Nombre," +
                                    " u.lastname AS Apellido" +
                                    " FROM" +
                                    " mdl_course c" +
                                    " INNER JOIN mdl_context ctx ON (c.id = ctx.instanceid)" +
                                    " INNER JOIN mdl_course_categories p ON (c.category = p.id)" +
                                    " INNER JOIN mdl_course_categories cr ON (p.parent = cr.id)" +
                                    " INNER JOIN mdl_course_categories s ON (cr.parent = s.id)" +
                                    " INNER JOIN mdl_role_assignments ra ON (ctx.id = ra.contextid)" +
                                    " INNER JOIN mdl_role r ON (ra.roleid = r.id)" +
                                    " INNER JOIN mdl_user u ON (ra.userid = u.id)" +
                                    " WHERE" +
                                    " s.id BETWEEN 2 AND 4 AND " +
                                    " c.visible = 1 AND " +
                                    " r.id LIKE 3" +
                                    " GROUP BY" +
                                    " p.name," +
                                    " c.id," +
                                    " c.fullname," +
                                    " c.shortname" +
                                    " ORDER BY " +
                                    "   s.id, " +
                                    "   cr.name ";

            con.comando = new MySqlCommand(nombreProfes, con.conex);
            profes = con.comando.ExecuteReader();

            List<ClassProfe> resultadosProfes = new List<ClassProfe>();
            while (profes.Read()) {
                ClassProfe aux = new ClassProfe();
                aux.idCurso = profes["idCurso"].ToString();
                aux.carrera = profes["carrera"].ToString();
                aux.asignatura = profes["asignatura"].ToString();
                aux.Nombre = profes["Nombre"].ToString();
                aux.Apellido = profes["Apellido"].ToString();

                resultadosProfes.Add(aux);
            }

            MySqlDataReader alumnosTotal;
            Conexion con2 = new Conexion();
            
            string conteoTotalAlumnos = " SELECT "+
                                        " c.id AS idCurso, " +
                                        " p.name AS carrera, "+
                                        " c.id AS idCurso, "+
                                        " c.fullname AS asignatura, "+
                                        " c.shortname AS codigo, "+ 
                                        " count(distinct u.id) AS totalEstudiantes "+
                                        " FROM "+
                                        "   mdl_user u "+
                                        "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) "+
                                        "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) "+
                                        "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) "+
                                        "   INNER JOIN mdl_role r ON (ra.roleid = r.id) "+
                                        "   INNER JOIN mdl_course_categories p ON (c.category = p.id) "+
                                        "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) "+
                                        "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) "+
                                        " WHERE "+
                                        "   s.id BETWEEN 2 AND 4 AND  "+
                                        "   c.visible LIKE 1 AND  "+
                                        "   r.id LIKE '5' "+
                                        " GROUP BY "+
                                        "   p.name, "+
                                        "   c.id, "+
                                        "   c.fullname, " +
                                        "   c.shortname";


            con2.comando = new MySqlCommand(conteoTotalAlumnos, con2.conex);
            alumnosTotal = con2.comando.ExecuteReader();

            List<ClassEstudiantes> alumnosTotalLista = new List<ClassEstudiantes>();
            while (alumnosTotal.Read())
            {
                ClassEstudiantes aux = new ClassEstudiantes();
                aux.idCurso = alumnosTotal["idCurso"].ToString();
                aux.totalEstudiantes = alumnosTotal["totalEstudiantes"].ToString();

                alumnosTotalLista.Add(aux);
            }

            MySqlDataReader alumnosActivosReader;
            Conexion con3 = new Conexion();
            string alumnosActivos = " SELECT " +
                                    " c.id AS idCurso, " +
                                    " c.fullname AS asignatura, " +
                                    " count(distinct u.id) AS estudiantesActivos " +
                                    " FROM " +
                                    "   mdl_user u " +
                                    "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                                    "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                                    "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                                    "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                                    "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                                    "   AND (l.course = c.id) " +
                                    "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                                    "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                                    "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                                    " WHERE "+
                                    "   NOT l.module LIKE 'course' AND  "+
                                    "   NOT l.module LIKE 'user' AND  "+
                                    "   NOT l.module LIKE 'role' AND  "+
                                    Global.fecha +
                                    "   s.id BETWEEN 2 AND 4 AND  "+
                                    "   c.visible LIKE '1' AND  "+
                                    "   r.id LIKE '5' "+
                                    " GROUP BY "+
                                    "   c.id, "+
                                    "   c.fullname "+
                                    " ORDER BY "+
                                    "   s.id, "+
                                    "   cr.name ";

            con3.comando = new MySqlCommand(alumnosActivos, con3.conex);
            alumnosActivosReader = con3.comando.ExecuteReader();

            List<ClassEstudiantesActivos> alumnosActivosReaderLista = new List<ClassEstudiantesActivos>();
            while (alumnosActivosReader.Read())
            {
                ClassEstudiantesActivos aux = new ClassEstudiantesActivos();
                aux.idCurso = alumnosActivosReader["idCurso"].ToString();
                aux.estudiantesActivos = alumnosActivosReader["estudiantesActivos"].ToString();

                alumnosActivosReaderLista.Add(aux);
            }


            MySqlDataReader clickEstudiantesReader;
            Conexion con31 = new Conexion();
            string clickestudiantes = " SELECT " +
                                    " c.id AS idCurso, " +
                                    " c.fullname AS asignatura, " +
                                    " count(u.id) AS clickEstudiantes " +
                                    " FROM " +
                                    "   mdl_user u " +
                                    "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                                    "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                                    "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                                    "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                                    "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                                    "   AND (l.course = c.id) " +
                                    "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                                    "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                                    "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                                    " WHERE " +
                                    "   NOT l.module LIKE 'course' AND  " +
                                    "   NOT l.module LIKE 'user' AND  " +
                                    "   NOT l.module LIKE 'role' AND  " +
                                    Global.fecha +
                                    "   s.id BETWEEN 2 AND 4 AND  " +
                                    "   c.visible LIKE '1' AND  " +
                                    "   r.id LIKE '5' " +
                                    " GROUP BY " +
                                    "   c.id, " +
                                    "   c.fullname " +
                                    " ORDER BY " +
                                    "   s.id, " +
                                    "   cr.name ";

            con31.comando = new MySqlCommand(clickestudiantes, con31.conex);
            clickEstudiantesReader = con31.comando.ExecuteReader();

            List<ClassClickEstudiantes> clickEstudiantesReaderLista = new List<ClassClickEstudiantes>();
            while (clickEstudiantesReader.Read())
            {
                ClassClickEstudiantes aux = new ClassClickEstudiantes();
                aux.idCurso = clickEstudiantesReader["idCurso"].ToString();
                aux.clickEstudiantes = clickEstudiantesReader["clickEstudiantes"].ToString();
                //aux.estudiantesActivos = clickEstudiantesReader["estudiantesActivos"].ToString();
                clickEstudiantesReaderLista.Add(aux);
            }

            MySqlDataReader clickProfesoresReader;
            Conexion con4 = new Conexion();
            string clicksProfes =   " SELECT  "+
                                    " c.id AS idCurso, " +
                                    " c.fullname AS asignatura, "+
                                   // " count(distinct u.id) AS totalProfesores, "+
                                    " count(l.id) AS clickProfesores " +
                                    " FROM "+
                                    "   mdl_user u "+
                                    "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) "+
                                    "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) "+
                                    "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) "+
                                    "   INNER JOIN mdl_role r ON (ra.roleid = r.id) "+
                                    "   INNER JOIN mdl_log l ON (u.id = l.userid) "+
                                    "   AND (l.course = c.id) "+
                                    "   INNER JOIN mdl_course_categories p ON (c.category = p.id) "+
                                    "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) "+
                                    "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) "+
                                    " WHERE "+
                                    "   NOT l.module LIKE 'course' AND  " +
                                    "   NOT l.module LIKE 'user' AND  " +
                                    "   NOT l.module LIKE 'role' AND  " +
                                    Global.fecha +
                                    "   s.id BETWEEN 2 AND 4 AND  "+
                                    "   c.visible LIKE 1 AND  "+
                                    "   r.id LIKE '3' "+
                                    " GROUP BY " +
                                    "   c.id, " +
                                    "   c.fullname " +
                                    " ORDER BY " +
                                    "   s.id, " +
                                    "   cr.name ";
            
            con4.comando = new MySqlCommand(clicksProfes, con4.conex);
            clickProfesoresReader = con4.comando.ExecuteReader();

            List<ClassClickProfesores> clickProfesoresReaderLista = new List<ClassClickProfesores>();
            while (clickProfesoresReader.Read())
            {
                ClassClickProfesores aux = new ClassClickProfesores();
                aux.idCurso = clickProfesoresReader["idCurso"].ToString();
                aux.clickProfesores = clickProfesoresReader["clickProfesores"].ToString();

                clickProfesoresReaderLista.Add(aux);
            }

            MySqlDataReader tareaReader;
            Conexion con5 = new Conexion();
            string tar = " SELECT "+
                            " c.id AS idCurso, "+
                            " c.fullname AS asignatura, "+
                            " count(l.action) AS totalTarea "+
                            " FROM "+
                            "   mdl_user u "+
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) "+
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) "+
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) "+
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) "+
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) "+
                            "   AND (l.course = c.id) "+
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) "+
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) "+
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) "+
                            " WHERE "+
                            "   NOT l.module LIKE 'course' AND  "+
                            "   NOT l.module LIKE 'user' AND  "+
                            "   NOT l.module LIKE 'role' AND  "+
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  "+
                            "   c.visible LIKE '1' AND  "+
                            "   r.id LIKE '5' AND  "+
                            "   l.module = 'assign'  "+
                            " GROUP BY "+
                            "   c.id, "+
                            "   c.fullname "+
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con5.comando = new MySqlCommand(tar, con5.conex);
            tareaReader = con5.comando.ExecuteReader();

            List<ClassTareas> tareaReaderLista = new List<ClassTareas>();
            while (tareaReader.Read())
            {
                ClassTareas aux = new ClassTareas();
                aux.idCurso = tareaReader["idCurso"].ToString();
                aux.totalTarea = tareaReader["totalTarea"].ToString();

                tareaReaderLista.Add(aux);
            }

            MySqlDataReader blogReader;
            Conexion con6 = new Conexion();
            string blogs = " SELECT " +
                        " c.id AS idCurso, " +
                        " c.fullname AS asignatura, " +
                        " count(l.action) AS totalBlog " +
                        " FROM " +
                        "   mdl_user u " +
                        "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                        "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                        "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                        "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                        "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                        "   AND (l.course = c.id) " +
                        "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                        "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                        "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                        " WHERE " +
                        "   NOT l.module LIKE 'course' AND  " +
                        "   NOT l.module LIKE 'user' AND  " +
                        Global.fecha +
                        "   NOT l.module LIKE 'role' AND  " +
                        "   s.id BETWEEN 2 AND 4 AND  " +
                        "   c.visible LIKE '1' AND  " +
                        "   r.id LIKE '5' AND  " +
                        "   l.module = 'blog'  " +
                        " GROUP BY " +
                        "   c.id, " +
                        "   c.fullname " +
                        " ORDER BY " +
                        "   s.id, " +
                        "   cr.name ";
            con6.comando = new MySqlCommand(blogs, con6.conex);
            blogReader = con6.comando.ExecuteReader();

            List<ClassBlog> blogReaderLista = new List<ClassBlog>();
            while (blogReader.Read())
            {
                ClassBlog aux = new ClassBlog();
                aux.idCurso = blogReader["idCurso"].ToString();
                aux.totalBlog = blogReader["totalBlog"].ToString();

                blogReaderLista.Add(aux);
            }

            string fecha = "";
            MySqlDataReader carpetaReader;
            Conexion con7 = new Conexion();
            string carp = " SELECT "+
                            " c.id AS idCurso, "+
                            " c.fullname AS asignatura, "+
                            " count(l.action) AS totalCarpeta "+
                            " FROM "+
                            "   mdl_user u "+
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) "+
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) "+
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) "+
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) "+
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) "+
                            "   AND (l.course = c.id) "+
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) "+
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) "+
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) "+
                            " WHERE "+
                            "   NOT l.module LIKE 'course' AND  "+
                            "   NOT l.module LIKE 'user' AND  "+
                            "   NOT l.module LIKE 'role' AND  "+
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  "+
                            "   c.visible LIKE '1' AND  "+
                            "   r.id LIKE '5' AND  "+
                            "   l.module = 'folder'  "+
                            fecha+
                            " GROUP BY "+
                            "   c.id, "+
                            "   c.fullname "+
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con7.comando = new MySqlCommand(carp, con7.conex);
            carpetaReader = con7.comando.ExecuteReader();

            List<ClassCarpeta> carpetaReaderLista = new List<ClassCarpeta>();
            while (carpetaReader.Read())
            {
                ClassCarpeta aux = new ClassCarpeta();
                aux.idCurso = carpetaReader["idCurso"].ToString();
                aux.totalCarpeta = carpetaReader["totalCarpeta"].ToString();

                carpetaReaderLista.Add(aux);
            }

            MySqlDataReader foroReader;
            Conexion con8 = new Conexion();
            string foros = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalForo " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'forum'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con8.comando = new MySqlCommand(foros, con8.conex);
            foroReader = con8.comando.ExecuteReader();

            List<ClassForo> foroReaderLista = new List<ClassForo>();
            while (foroReader.Read())
            {
                ClassForo aux = new ClassForo();
                aux.idCurso = foroReader["idCurso"].ToString();
                aux.totalForo = foroReader["totalForo"].ToString();

                foroReaderLista.Add(aux);
            }

            MySqlDataReader paginaReader;
            Conexion con9 = new Conexion();
            string paginas = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalPaginas " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'page'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con9.comando = new MySqlCommand(paginas, con9.conex);
            paginaReader = con9.comando.ExecuteReader();

            List<ClassPagina> paginaReaderLista = new List<ClassPagina>();
            while (paginaReader.Read())
            {
                ClassPagina aux = new ClassPagina();
                aux.idCurso = paginaReader["idCurso"].ToString();
                aux.totalPagina = paginaReader["totalPaginas"].ToString();

                paginaReaderLista.Add(aux);
            }

            MySqlDataReader cuestionarioReader;
            Conexion con10 = new Conexion();
            string cuestion = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalCuestionario " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'questionnaire'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con10.comando = new MySqlCommand(cuestion, con10.conex);
            cuestionarioReader = con10.comando.ExecuteReader();

            List<ClassCuestionario> cuestionarioReaderLista = new List<ClassCuestionario>();
            while (cuestionarioReader.Read())
            {
                ClassCuestionario aux = new ClassCuestionario();
                aux.idCurso = cuestionarioReader["idCurso"].ToString();
                aux.totalCuestionario = cuestionarioReader["totalCuestionario"].ToString();

                cuestionarioReaderLista.Add(aux);
            }

            MySqlDataReader pruebaReader;
            Conexion con11 = new Conexion();
            string prueb = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalPrueba " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'quiz'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con11.comando = new MySqlCommand(prueb, con11.conex);
            pruebaReader = con11.comando.ExecuteReader();

            List<ClassPrueba> pruebaReaderLista = new List<ClassPrueba>();
            while (pruebaReader.Read())
            {
                ClassPrueba aux = new ClassPrueba();
                aux.idCurso = pruebaReader["idCurso"].ToString();
                aux.totalPrueba = pruebaReader["totalPrueba"].ToString();

                pruebaReaderLista.Add(aux);
            }

            MySqlDataReader recursoReader;
            Conexion con12 = new Conexion();
            string recurs = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalRecurso " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'resource'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con12.comando = new MySqlCommand(recurs, con12.conex);
            recursoReader = con12.comando.ExecuteReader();

            List<ClassRecurso> recursoReaderLista = new List<ClassRecurso>();
            while (recursoReader.Read())
            {
                ClassRecurso aux = new ClassRecurso();
                aux.idCurso = recursoReader["idCurso"].ToString();
                aux.totalRecurso = recursoReader["totalRecurso"].ToString();

                recursoReaderLista.Add(aux);
            }

            MySqlDataReader urlReader;
            Conexion con13 = new Conexion();
            string urls = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalURL " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'url'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con13.comando = new MySqlCommand(urls, con13.conex);
            urlReader = con13.comando.ExecuteReader();

            List<ClassURL> urlReaderLista = new List<ClassURL>();
            while (urlReader.Read())
            {
                ClassURL aux = new ClassURL();
                aux.idCurso = urlReader["idCurso"].ToString();
                aux.totalURL = urlReader["totalURL"].ToString();

                urlReaderLista.Add(aux);
            }

            MySqlDataReader workshopReader;
            Conexion con14 = new Conexion();
            string worksh = " SELECT " +
                            " c.id AS idCurso, " +
                            " c.fullname AS asignatura, " +
                            " count(l.action) AS totalWorkshop " +
                            " FROM " +
                            "   mdl_user u " +
                            "   INNER JOIN mdl_role_assignments ra ON (u.id = ra.userid) " +
                            "   INNER JOIN mdl_context ctx ON (ra.contextid = ctx.id) " +
                            "   INNER JOIN mdl_course c ON (ctx.instanceid = c.id) " +
                            "   INNER JOIN mdl_role r ON (ra.roleid = r.id) " +
                            "   INNER JOIN mdl_log l ON (u.id = l.userid) " +
                            "   AND (l.course = c.id) " +
                            "   INNER JOIN mdl_course_categories p ON (c.category = p.id) " +
                            "   INNER JOIN mdl_course_categories cr ON (p.parent = cr.id) " +
                            "   INNER JOIN mdl_course_categories s ON (cr.parent = s.id) " +
                            " WHERE " +
                            "   NOT l.module LIKE 'course' AND  " +
                            "   NOT l.module LIKE 'user' AND  " +
                            "   NOT l.module LIKE 'role' AND  " +
                            Global.fecha +
                            "   s.id BETWEEN 2 AND 4 AND  " +
                            "   c.visible LIKE '1' AND  " +
                            "   r.id LIKE '5' AND  " +
                            "   l.module = 'workshop'  " +
                            fecha +
                            " GROUP BY " +
                            "   c.id, " +
                            "   c.fullname " +
                            " ORDER BY " +
                            "   s.id, " +
                            "   cr.name ";
            con14.comando = new MySqlCommand(worksh, con14.conex);
            workshopReader = con14.comando.ExecuteReader();

            List<ClassWorkshop> workshopReaderLista = new List<ClassWorkshop>();
            while (workshopReader.Read())
            {
                ClassWorkshop aux = new ClassWorkshop();
                aux.idCurso = workshopReader["idCurso"].ToString();
                aux.totalWorkshop = workshopReader["totalWorkshop"].ToString();

                workshopReaderLista.Add(aux);
            }

            string numCurso = "";
            string asignatura = "";
            string profesor = "";
            string estudiantesTotal = "0";
            string estudiantesActivos = "0";
            string clickEstudiantes = "0";
            string clickProfe = "0";
            string tareas= "0";
            string bl = "0";
            string carpetas = "0";
            string foro = "0";
            string pagina = "0";
            string cuestionario = "0";
            string prueba = "0";
            string recurso = "0";
            string url = "0";
            string workshop = "0";
            tbl_datos.Rows.Clear();

            for (int i = 0; i < resultadosProfes.Count; i++)
            {
                numCurso = resultadosProfes[i].idCurso;
                asignatura = resultadosProfes[i].asignatura;
                profesor = resultadosProfes[i].Nombre +" " + resultadosProfes[i].Apellido;

                for (int j = 0; j < alumnosTotalLista.Count; j++)
                {
                    if (numCurso == alumnosTotalLista[j].idCurso)
                    {
                        estudiantesTotal = alumnosTotalLista[j].totalEstudiantes;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < alumnosActivosReaderLista.Count; j++)
                {
                    if (numCurso == alumnosActivosReaderLista[j].idCurso)
                    {
                        estudiantesActivos = alumnosActivosReaderLista[j].estudiantesActivos;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < clickEstudiantesReaderLista.Count; j++)
                {
                    if (numCurso == clickEstudiantesReaderLista[j].idCurso)
                    {
                        clickEstudiantes = clickEstudiantesReaderLista[j].clickEstudiantes;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < clickProfesoresReaderLista.Count; j++)
                {
                    if (numCurso == clickProfesoresReaderLista[j].idCurso)
                    {
                        clickProfe = clickProfesoresReaderLista[j].clickProfesores;
                        break;
                    }
                }//este es el que se repite
              
                for (int j = 0; j < tareaReaderLista.Count; j++)
                {
                    if (numCurso == tareaReaderLista[j].idCurso)
                    {
                        tareas = tareaReaderLista[j].totalTarea;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < blogReaderLista.Count; j++)
                {
                    if (numCurso == blogReaderLista[j].idCurso)
                    {
                        bl = blogReaderLista[j].totalBlog;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < carpetaReaderLista.Count; j++)
                {
                    if (numCurso == carpetaReaderLista[j].idCurso)
                    {
                        carpetas = carpetaReaderLista[j].totalCarpeta;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < foroReaderLista.Count; j++)
                {
                    if (numCurso == foroReaderLista[j].idCurso)
                    {
                        foro = foroReaderLista[j].totalForo;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < paginaReaderLista.Count; j++)
                {
                    if (numCurso == paginaReaderLista[j].idCurso)
                    {
                        pagina = paginaReaderLista[j].totalPagina;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < cuestionarioReaderLista.Count; j++)
                {
                    if (numCurso == cuestionarioReaderLista[j].idCurso)
                    {
                        cuestionario = cuestionarioReaderLista[j].totalCuestionario;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < pruebaReaderLista.Count; j++)
                {
                    if (numCurso == pruebaReaderLista[j].idCurso)
                    {
                        prueba = pruebaReaderLista[j].totalPrueba;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < recursoReaderLista.Count; j++)
                {
                    if (numCurso == recursoReaderLista[j].idCurso)
                    {
                        recurso = recursoReaderLista[j].totalRecurso;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < urlReaderLista.Count; j++)
                {
                    if (numCurso == urlReaderLista[j].idCurso)
                    {
                        url = urlReaderLista[j].totalURL;
                        break;
                    }
                }//este es el que se repite

                for (int j = 0; j < workshopReaderLista.Count; j++)
                {
                    if (numCurso == workshopReaderLista[j].idCurso)
                    {
                        workshop = workshopReaderLista[j].totalWorkshop;
                        break;
                    }
                }//este es el que se repite

                tbl_datos.Rows.Add(numCurso, asignatura, profesor, estudiantesTotal, estudiantesActivos, clickProfe, clickEstudiantes, tareas, bl, carpetas, foro, pagina, cuestionario, prueba, recurso, url, workshop);

                estudiantesTotal = "0";
                estudiantesActivos = "0";
                clickEstudiantes = "0";
                clickProfe = "0";
                tareas = "0";
                bl = "0";
                carpetas = "0";
                foro = "0";
                pagina = "0";
                cuestionario = "0";
                prueba = "0";
                recurso = "0";
                url = "0";
                workshop = "0";
                
            }

        }

        public class ClassProfe
        {
            public string idCurso { get; set; }
            public string carrera { get; set; }
            public string asignatura { get; set; }
            public string Nombre { get; set; }
            public string Apellido { get; set; }            
        }

        public class ClassEstudiantes
        {
            public string idCurso { get; set; }
            public string carrera { get; set; }
            public string asignatura { get; set; }
            public string codigo { get; set; }
            public string totalEstudiantes { get; set; }
        }

        public class ClassEstudiantesActivos
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string estudiantesActivos { get; set; }
            public string clickEstudiantes { get; set; }            
        }
   
        public class ClassClickProfesores
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string totalProfesores { get; set; }
            public string clickProfesores { get; set; }            
        }

        public class ClassClickEstudiantes
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string clickEstudiantes { get; set; }
        }

        public class ClassTareas
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string totalTarea { get; set; }
        }
        public class ClassBlog
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string totalBlog { get; set; }
        }

         public class ClassCarpeta
        {
            public string idCurso { get; set; }
            public string asignatura { get; set; }
            public string totalCarpeta { get; set; }
        }

         public class ClassForo
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalForo { get; set; }
         }

         public class ClassPagina
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalPagina { get; set; }
         }

         public class ClassCuestionario
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalCuestionario { get; set; }
         }

         public class ClassPrueba
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalPrueba { get; set; }
         }

         public class ClassRecurso
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalRecurso { get; set; }
         }

         public class ClassURL
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalURL { get; set; }
         }

         public class ClassWorkshop
         {
             public string idCurso { get; set; }
             public string asignatura { get; set; }
             public string totalWorkshop { get; set; }
         }

        private void pdf_btn_Click(object sender, EventArgs e)
        {
            Document doc = new Document(iTextSharp.text.PageSize.LETTER.Rotate());
            iTextSharp.text.pdf.PdfWriter writer = iTextSharp.text.pdf.PdfWriter.GetInstance(doc, new FileStream(@"C:\Users\usuario3\Desktop\Reporte\ReporteAula.pdf", FileMode.Create));

            doc.Open();
            doc.AddTitle("Reporte PDF");
            doc.AddCreator("Pilar Valdés Urrutia - pilar.v.urrutia@gmail.com");

            iTextSharp.text.Image imagen = iTextSharp.text.Image.GetInstance(@"C:\Users\usuario3\Desktop\snif\logo.png");

            var parrafo2 = new Paragraph("AULA VIRTUAL CFT SAN AGUSTÍN", FontFactory.GetFont(FontFactory.HELVETICA_BOLD));
            doc.Add(new Paragraph("REPORTE DE ACTIVIDAD DE MÓDULOS", FontFactory.GetFont(FontFactory.HELVETICA_BOLD)));
            parrafo2.SpacingBefore = 1;
            parrafo2.SpacingAfter = 0;
            parrafo2.Alignment = 0; //0-Left, 1 middle,2 Right
            doc.Add(parrafo2);
            doc.Add(Chunk.NEWLINE);

            imagen.ScaleToFit(125f, 60F);
            imagen.SetAbsolutePosition(630, 530);
            doc.Add(imagen);

            PdfPTable table = new PdfPTable(tbl_datos.Columns.Count);
            table.WidthPercentage = 100;
            float[] medidaCeldas = { 0.55f, 2.25f, 2.25f, 0.4f, 0.55f, 0.6f, 0.7f, 0.45f, 0.4f, 0.5f, 0.4f, 0.5f, 0.75f, 0.5f, 0.6f, 0.4f, 0.65f };
            table.SetWidths(medidaCeldas);

            for (int j = 0; j < tbl_datos.Columns.Count; j++)
            {
                Paragraph p = new Paragraph(tbl_datos.Columns[j].HeaderText, FontFactory.GetFont(FontFactory.HELVETICA_BOLD, 5));
                PdfPCell cell = new PdfPCell(p);
                cell.BorderWidthBottom = 0.75f;
                cell.VerticalAlignment = Element.ALIGN_MIDDLE;
                cell.HorizontalAlignment = Element.ALIGN_CENTER;
                table.AddCell(cell);                               
            }

            table.HeaderRows = 1;
            for (int i = 0; i < tbl_datos.Rows.Count; i++)
            {
                for (int k = 0; k < tbl_datos.Columns.Count; k++)
                {
                    

                    if (tbl_datos[k,i].Value !=null)
                    {
                        Paragraph pa = new Paragraph(tbl_datos[k, i].Value.ToString(), FontFactory.GetFont("Arial", 8));

                        PdfPCell paf = new PdfPCell(pa);
                        paf.BorderWidthBottom = 0.75f;
                        paf.VerticalAlignment = Element.ALIGN_MIDDLE;
                        if (k == 1 || k == 2)
                        {
                            paf.HorizontalAlignment = Element.ALIGN_LEFT;
                        }
                        else 
                        {
                            paf.HorizontalAlignment = Element.ALIGN_CENTER;
                        }
                        table.AddCell(paf);
                    }

                }
            }

            doc.Add(table);
            doc.Close();
            MessageBox.Show("Su reporte ha sido creado con éxito");
        }

        private void button1_Click(object sender, EventArgs e)
        {
          //  string mes = cmb_filtro.SelectedItem.ToString();
          //  if (mes == "Diciembre") {
           //     Global.fecha = " l.time BETWEEN 1477969200 AND 1480561199";

           // }

            Global.fecha = " l.time BETWEEN 1477969200 AND 1480561199 AND "; //valor noviembre por defecto
            MessageBox.Show("Su reporte se generará en unos momentos");
            rellenar();
            
        }

        private void tbl_datos_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {

        }

        private void cmb_filtro_SelectedIndexChanged(object sender, EventArgs e)
        {

        }    
     
    }
}

