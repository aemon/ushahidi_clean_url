ushahidi_clean_url
==================

Make the clean URL in ushahidi

Useful for those, who wants to make clean URL.

Below is link for thread on ushahidi forum. 
https://wiki.ushahidi.com/pages/viewpage.action?pageId=8359093





And here is original post without comments:

I made it. For those, who want to do the same thing.
 
In file application/config/routes.php add these lines:
// for SEF-reports
$config['reports/([0-9]+)(-)([a-z0-9/-]+)'] = 'reports/view/$1';
$config['reports/([0-9]+)-'] = 'reports/view/$1';
$config['reports/([0-9]+)'] = 'reports/view/$1';

last 2 strings helps us correct wrong lines.
In application/helpers/url.php (if you don't have this file just copy it from system/helpers):
in function site after 
if ($path = trim(parse_url($uri, PHP_URL_PATH), '/'))
        {
            // Add path suffix
            $path .= Kohana::config('core.url_suffix');
        }

add 
        // for SEF-reports
        if (strpos($path,'reports/view/')!==false){
            
            $path = str_replace('reports/view/', 'reports/', $path);
            $report_id =  intval(substr($path, strlen('reports/'), strlen($path)));
            if ($report_id){
                $report = ORM::factory('incident')->where('id', $report_id)->find();
                if (!empty($report->incident_title)){
                    $translit_report_title = url::translateRuToRus($report->incident_title);
                    $report_title = url::title($translit_report_title);
                    if (!empty($report_title)){
                        $path .='-'.$report_title; 
                    }

                }
            }
        }
If you want translation for russian and ukrainian language then add next function at the end:

 
public static function translateRuToRus($str)
{
$tr = array(
"А"=>"A","Б"=>"B","В"=>"V","Г"=>"G",
"Д"=>"D","Е"=>"E","Ё"=>"E","Ж"=>"J","З"=>"Z","▒?"=>"I",
"Й"=>"Y","К"=>"K","Л"=>"L","М"=>"M","Н"=>"N",
"О"=>"O","П"=>"P","Р"=>"R","С"=>"S","Т"=>"T",
"У"=>"U","Ф"=>"F","Х"=>"H","Ц"=>"TS","Ч"=>"CH",
"Ш"=>"SH","Щ"=>"SCH","Ъ"=>"","Ы"=>"YI","Ь"=>"",
"Э"=>"E","Ю"=>"YU","Я"=>"YA","а"=>"a","б"=>"b",
"в"=>"v","г"=>"g","д"=>"d","е"=>"e","ё"=>"e","ж"=>"j",
"з"=>"z","и"=>"i","й"=>"y","к"=>"k","л"=>"l",
"м"=>"m","н"=>"n","о"=>"o","п"=>"p","р"=>"r",
"с"=>"s","т"=>"t","у"=>"u","ф"=>"f","х"=>"h",
"ц"=>"ts","ч"=>"ch","ш"=>"sh","щ"=>"sch","ъ"=>"y",
"ы"=>"yi","ь"=>"","э"=>"e","ю"=>"yu","я"=>"ya"," "=>"-", "-"=>"-","."=>"-","+"=>"-"
);
$str = strtr($str,$tr);
$str = preg_replace("/[^a-zA-Z0-9_\-]/","",$str);
return $str;
}
 
 
So, you can add your own translation array for different language. 

Important:
in application/controllers/scheduler/s_alerts.php change 
$incident_url = url::site().'reports/view/'.$incident->id; 
to
$incident_url = url::site('reports/view/'.$incident->id);
