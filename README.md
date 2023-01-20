# Moodle-WebServiceClient
PHP written code for creating attendance, session, and getting user information.

## Basic Functions Code:
```php
require_once "MoodleRest.php"; //MoodleRest API

$ip = '202.131.126.214'; //Moodle Server IP
$attendancetoken = '697859a828111d63c3f68543ac986827'; //Token for usage of mod_attendance
$coretoken = '4f3c9f8f0404a7db50825391c295937e'; //Token for usage of core files

$MoodleCoreRest = new MoodleRest();
$MoodleCoreRest->setServerAddress("http://$ip/webservice/rest/server.php");
$MoodleCoreRest->setToken($coretoken);
$MoodleCoreRest->setReturnFormat(MoodleRest::RETURN_JSON); //Returns everything in JSON format.
$MoodleCoreRest->setDebug();  //For Debugging Purpose and better view of error logs.


$MoodleAttendanceRest = new MoodleRest();
$MoodleAttendanceRest->setServerAddress("http://$ip/webservice/rest/server.php");
$MoodleAttendanceRest->setToken($attendancetoken);
$MoodleAttendanceRest->setReturnFormat(MoodleRest::RETURN_JSON); //Returns everything in JSON format.
$MoodleAttendanceRest->setDebug(); //For Debugging Purpose and better view of error logs.


//Used for passing the hidden information to every student which contains faculty location for condition match!
$userid = 2; //FRONT END TEAM INPUT
$facultyname = 'something'; //FRONT END TEAM INPUT
$facloc = 'coordinateshere'; //FRONT END TEAM INPUT
$sessionid = 'somesessionidhere'; //FRONT END TEAM INPUT
$starttime = "timehere"; //FRONT END TEAM INPUT
$endtime = "timehere"; //FRONT END TEAM INPUT

$sendMessageArray = array('faculty' => $facultyname, 'facultyloc' => $facloc, 'sessionid'=> $sessionid, 'starttime'=>$starttime, 'exptime'=>$endtime); //Creating array of sending message, so it's easy to access in different app.

$messageencoded = json_encode($sendMessageArray, JSON_FORCE_OBJECT); //Encoding Message in json format, and decode it when we accessing it.
$sendMessage = $MoodleCoreRest->request('core_message_send_instant_messages', array('messages'=> array(array('touserid' => $userid, 'text' => $messageencoded))));
$sentmsgdecode = json_decode($sendMessage);

function getv($msg, $find){ //It is a function used to get value from the object.
    return get_object_vars($msg[0])[$find];
}

function getsv($msg, $find){ //It is a function used to get value from the object.
    return get_object_vars($msg)[$find];
}


$getMsgID = getv($sentmsgdecode, 'msgid'); //Must be saved.
$getConvoID = getv($sentmsgdecode, 'conversationid'); //Must be saved.
$getFromUserID = getv($sentmsgdecode, 'useridfrom'); //Must be saved.

//For reading the message from the convo.
$getMessage = $MoodleCoreRest->request('core_message_get_messages', array('useridto'=>$userid, 'type' => 'conversations', 'read'=> 2));
$getMsgDecode = json_decode($getMessage);
$getMsg = getsv($getMsgDecode, 'messages');
$fullMessage = getv($getMsg, 'fullmessage');
$trimFullMsg = trim($fullMessage, '{}'); //Triming { and } in the message
$finalMsg = explode(',', $trimFullMsg); //Creating the string in array with seperator ","
$faculty = $finalMsg[0]; //Returns "faculty":"something"
$facultyloc = $finalMsg[1]; //Returns "facultyloc":"coordinateshere"
$session = $finalMsg[2]; //Returns "sessionid":"somesessionidhere"


//This will be used for login in student app. In login, it will match if the given input of username is existing in moodle or not. RETURNS BOOL!
$usernameinput = "vrp"; //INPUT BY FRONT END TEAM
$getUsername = $MoodleCoreRest->request('core_user_get_users_by_field', array('field' => 'username', 'values' => array($usernameinput)));
$usernameData = json_decode($getUsername, true);
$checkusername = false;
if(!empty($usernameData)) { //Before checks, if array is empty or not. If it is empty, no use to go deeper inside the array.
    $getUserUsername = $usernameData["0"]["username"]; //Fetching Username in fetched array.
    if($getUserUsername == $usernameinput) { // MATCH CONDITION
        $checkusername = true; //This means, the username exists in moodle and is allowed to go inside attendance.
    }
}
//var_export($checkusername, true); //This is used to return bool in string format i.e TRUE/FALSE

//For fetching user id
$usernametosearch = "vrp"; //FRONT END TEAM INPUT
$getUserID = $MoodleCoreRest->request('core_user_get_users_by_field', array('field' => 'username', 'values' => array($usernametosearch)));
$userData = json_decode($getUserID, true);
$fetchedUserID = $userData["0"]["id"];

//Get list of courses the user is enrolled with userid
$inputUserID = "5"; //FRONT END TEAM INPUT
$selectedCourseName = "BDA_7CEIT-A_VRP_PMS_MDT"; //FRONT END TEAM INPUT
$getListOfCourseEnrolled = $MoodleCoreRest->request('core_enrol_get_users_courses', array('userid' => $fetchedUserID));
$courseData = json_decode($getListOfCourseEnrolled, true);
$courseID = 0;

//Loop the whole array and match the selected course name and grab its course id:
foreach ($courseData as $key => $value) {
    if($value['fullname'] == $selectedCourseName)
    {
        $courseID = $value['id']; //Setting the course id.
    }
}


//For creating attendance table with generating session id.
$courseid = "13"; //Course ID fetched from above.
$createAttendance = $MoodleAttendanceRest->request('mod_attendance_add_attendance', array('courseid' => $courseid, 'name'=> "demoattendance"));
$attendanceid = createAttendance["attendanceid"];

//$deleteAttendance = $MoodleAttendanceRest->request('mod_attendance_remove_attendance', array('attendanceid' => 10)); //Used for deleting attendace table, used for dev purpose only.

$createSession = $MoodleAttendanceRest->request('mod_attendance_add_session', array('attendanceid' => $attendanceid, 'sessiontime' => 'sessionstarttimestamphere'));
$sessionid = $createSession["sessionid"]; //Important is session id, used for setting absent and present in attendance.



//For changing attendance...
//Present - Status ID: 21
//Absent - Status ID: 22
$courseTakenByID = "326"; //ID OF THE ASIGNED TEACHER OF THE COURSE
$changeattendance = $MoodleAttendanceRest->request('mod_attendance_update_user_status', array('sessionid' => $sessionid, 'studentid' => $fetchedUserID, 'takenbyid' => $courseTakenByID, 'statusid' => 22, 'statusset' => 21));

// ^ This request is used for update attendance either absent or present on moodle.
```
