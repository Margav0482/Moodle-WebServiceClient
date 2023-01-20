<?php
 $domainname = 'http://202.131.126.214/';
 $restformat = 'json';
require_once('curl.php'); //Download this file
$curl = new curl;

$newparams = array(
    'draftitemid' => 641595481,
    'userid' => 2
);
$restformat = ($restformat == 'json') ?
'&moodlewsrestformat=' . $restformat : '';
$setprofileserverurl = $domainname . '/webservice/rest/server.php' . '?wstoken=4f3c9f8f0404a7db50825391c295937e' . '&wsfunction=core_user_update_picture';
$resp1 = $curl->post($setprofileserverurl . $restformat, $newparams);
$resps1 = json_decode($resp1);

print_r($resps1);
printf("\n");
