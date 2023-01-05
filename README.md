# Moodle-WebServiceClient
PHP written code for creating attendance, session, and getting user information.

## MAIN FUNCTION CODE:
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

For fetching user id
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

$createSession = $MoodleAttendanceRest->request('mod_attendance_add_session', array('attendaceid' => $attendanceid));
$sessionid = $createSession["sessionid"]; //Important is session id, used for setting absent and present in attendance.



//For changing attendance...
//Present - Status ID: 21
//Absent - Status ID: 22
$courseTakenByID = "326"; //ID OF THE ASIGNED TEACHER OF THE COURSE
$changeattendance = $MoodleAttendanceRest->request('mod_attendance_update_user_status', array('sessionid' => $sessionid, 'studentid' => $fetchedUserID, 'takenbyid' => $courseTakenByID, 'statusid' => 22, 'statusset' => 21));

// ^ This request is used for update attendance either absent or present on moodle.
```
