# JavaScript-and-WebSql
Websql work with Javascript
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title></title>
  <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" ></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.11.0/umd/popper.min.js" > </script>
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/js/bootstrap.min.js"> </script>
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta/css/bootstrap.min.css">
  <style type="text/css">
  table {
    font-family: arial, sans-serif;
    border-collapse: collapse;
    width: 100%;
  }

  td, th {
    border: 1px solid #dddddd;
    text-align: left;
    padding: 8px;
  }

  tr:nth-child(even) {
    background-color: #dddddd;
  }
</style>

</head>
<body>
  <table class="table table-hover">
    <div >
      <input type="text" class="bg-warning" id="name" placeholder="Enter your name">
      <input type="text" class="bg-warning" id="surName" placeholder="Enter your surname">
      <input id="rowAdd" class="btn btn-primary" type="button" value="Ekle">
    </div>
  </table>
  <table >
    <thead  >
      <tr>
        <th>ID</th>
        <th> Name</th>
        <th> Sur_Name</th>
        <th>Options</th>
      </tr>
    </thead>
    <tbody id="tbl_user_data">
    </tbody>
  </table>
</body>

<script>
  $(document).ready(function($){
    
//************VERİTABANI-ISLEMLERİ**********
if (window.openDatabase) {
        // VERİTABANINI OLUŞTUR
        var veritabani = openDatabase('person', '1.0', 'Web SQL Veritabanı', 4*1024*1024);  
        console.log("local_veritabani isimli veritabanı 1.0 versiyonu ile 10MB olacak şekilde oluşturuldu veya zaten varsa yeniden oluşturulmadı!");
        // KAYITLARI ÇEK VE LİSTELE
        kayitlariOku();
        tabloAdd();
      }else{
       alert("Maalesef tarayıcınızda Web SQL desteği bulunmamaktadır.")
     }
     function tabloAdd(){

      veritabani.transaction(function(tx) {
        tx.executeSql('CREATE TABLE persons(id INTEGER PRIMARY KEY, ad VARCHAR(35), soyad VARCHAR(25) )', [], function(islem, sonuc) {
          console.log(sonuc);
          console.log('Persons tablosu oluşturuldu.');
        });
      });
    }
//********************************

$(document).on('click', '#rowAdd',function(event){
//*************************************************
//?***********VeriTabanına kaydetme***************************
veritabani.transaction(function(tx) {
            // JQUERY İLE INPUT DEĞERLERİ ALINIYOR
            var name = $("#name").val();
            var surName = $("#surName").val();

            // EĞER İKİ DEĞER DE GİRİLMİŞSE...
            if(name && surName){
                // VERİLER GİRİLİYOR
                tx.executeSql('INSERT INTO persons (ad,soyad) VALUES (?,?)', [name, surName], function(islem, sonuc) {
                  console.log(sonuc);
                    // INPUT'LAR SIFIRLANIYOR
                    $("#name").val("");
                    $("#surName").val("");
                    console.log("isim kaydedildi...");
                    location.reload();
                    // KAYITLAR TEKRAR OKUNARAK TABLOYA AKTARILIYOR
                    
                  }, function(islem, hata) {
                    console.log("Hata: ", hata);
                  });
              }
            });
});
//***************************************************
//****************************************************
function kayitlariOku(){
  veritabani.transaction(function(tx) {
    tx.executeSql('SELECT * FROM persons', [], function(islem, sonuc) {
      console.log("Kayıtlar listeleniyor:")
      console.log(sonuc.rows); 
      $("#tbl_user_data").empty();
      jQuery.each(sonuc.rows, function(index, value) {
        var tbl = '';
      //  alert(value.id+" ** "+value.ad+" ** "+value.soyad);
      tbl +='<tr>  ';
      tbl +='<td>'+value.id+'</td>';
      tbl +='<td ><div class="row_data" id="name'+value.id+'"  edit_type="click" >'+value.ad+'</div></td>';
      tbl +='<td ><div class="row_data" id="surName'+value.id+'"  edit_type="click" >'+value.soyad+'</div></td>';
      tbl +='<td>';
      tbl +='<button type="button" class="btn_edit"  href="#" data-index="'+value.id+'" class="btn btn-link " row_id="'+value.id+'"> Edit</button> ';
      tbl +='<button type="button" class="btn_update" href="#" data-index="'+value.id+'" class="btn btn-link"  row_id="'+value.id+'">Update</button>';           
      tbl +='<button class="btn_delete" href="#" data-index="'+value.id+'" class="btn btn-link"  row_id="'+value.id+'"> Delete</button>';
      tbl +='</td>';
      tbl +='</tr>';
      $("#tbl_user_data").append(tbl);

    });
      $(document).find('.btn_update').hide();
      $(document).find('.btn_delete').hide();

    },function(islem, hata) {
      console.log("Tabo Yok")
      console.log("Hata: ", hata);
    });
  });
}
$(document).on('click', '.row_data', function(event) 
{
  event.preventDefault(); 

  if($(this).attr('edit_type') == 'button')
  {
    return false; 
  }
  $(this).closest('div').attr('contenteditable', 'false');

  $(this).addClass('bg-warning').css('padding','');

  $(this).focus();
})  ;
$(document).on('focusout', '.row_data', function(event) 
{
  event.preventDefault();

  if($(this).attr('edit_type') == 'button')
  {
    return false; 
  }
  var row_id = $(this).closest('tr').attr('row_id'); 

  var row_div = $(this)       
  .removeClass('bg-warning') 
  .css('padding','');
})  ;
$(document).on('click', '.btn_edit', function(event) 
{
  event.preventDefault();
  var tbl_row = $(this).closest('tr');
  var row_id = tbl_row.attr('row_id');
  tbl_row.find('.btn_update').show();
  tbl_row.find('.btn_delete').show();
  tbl_row.find('.btn_edit').hide(); 
  tbl_row.find('.row_data')
  .attr('contenteditable', 'true')
  .attr('edit_type', 'button');
});

$(document).on('click', '.btn_update', function(event) 
{
  event.preventDefault();
  var tbl_row = $(this).closest('tr');

  var row_id = tbl_row.attr('row_id');


  var index = $(this).attr('data-index');
  console.log("indexx :: "+index);
  var ad= $("#name"+index).html();
  var soyad = $("#surName"+index).html();
       // console.log(index+"--"+name+"--"+surName);
       console.log(ad+" ** "+soyad);
       if(ad && soyad){
         veritabani.transaction(function(tx) {
          tx.executeSql(SELECT )
          tx.executeSql('UPDATE persons SET ad=?, soyad=? WHERE id=?', [ad,soyad,index], function(islem, sonuc) {
            console.log(sonuc);
            console.log('Kayıt güncellendi.');
                    // GÜNCELLEME SONRASI KAYITLAR TEKRAR OKUNUYOR
                    kayitlariOku();                
                  }, function(islem, hata) {
                    console.log("Hata: ", hata);
                  });
        });
       }
       tbl_row.find('.btn_update').hide();
       $(document).find('.btn_delete').hide();
       tbl_row.find('.btn_edit').show();
       tbl_row.find('.row_data')
       .attr('edit_type', 'click')
       .removeClass('bg-warning');
     });
    $(document).on('click','.btn_delete',function(){
        // HANGİ ELEMENT TIKLANMIŞSA ONDAKİ data-index'E GÖRE INDEX OKUNUYOR
        var index = $(this).attr('data-index');
        veritabani.transaction(function(tx) {
            /* OKUNAN index DEĞERİ İLE VERİTABANINDAKİ id DEĞERİ EŞİT OLAN KAYIT SİLİNİYOR */
            tx.executeSql('DELETE FROM persons WHERE id = ?', [index], function(islem, sonuc) {
                console.log(sonuc);
                console.log('Kayıt silindi.');
                // KAYITLAR TEKRAR GÜNCELLENİYOR
                kayitlariOku();
            }, function(islem, hata) {
                console.log("Hata: ", hata);
            });
        });
 
    });
}); 
</script>
</html>
