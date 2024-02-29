# Mongodb-ZenclassProgramTask

//The Commands which i Executed Here the commands i executed for each Questions separately:

//1.Find all the topics and tasks which are thought in the month of October

db.getCollection('Topics').aggregate(
    [
      {
        $lookup: {
          from: 'tasks',
          localField: 'topics',
          foreignField: 'tasks',
          as: 'Topic Task Data'
        }
      },
      {
          $unwind:
          {
              path: "$Topic Task Data",
          }
      },
      {
        $match: { topicDate: { $regex: '2023-10' } }
      },
      {
        $project: {
          _id: 0,
          topic: 1,
          topicDate: 1,
          'Topic Task Data.userId': 1,
          'Topic Task Data.submitted': 1,
          'Topic Task Data.task': 1
        }
      }
    ]);

//2.Find all the company drives which appeared between 15 oct-2023 and 31-oct-2023

db.getCollection('Company_Drives').find({
    driveDate: {
      $gte: '2023-10-15',
      $lte: '2023-10-31'
    }
  });

//3.Find all the company drives and students who are appeared for the placement.

db.getCollection('Company_Drives').aggregate(
    [
      {
        $lookup: {
          from: 'users',
          localField: 'userId',
          foreignField: 'userId',
          as: 'studentInfo'
        }
      },
      {
          $unwind:
          {
              path: "$StudentInfo",
            }
      },
      {
        $project: {
          _id: 0,
          'studentInfo.userName': 1,
          company: 1,
          driveDate: 1,
          'studentInfo.userEmail': 1,
          'studentInfo.userId': 1
        }
      }
    ]);

//4.Find the number of problems solved by the user in codekata

db.getCollection('Codekata').aggregate(
    [
      {
        $group: {
          _id: 'Total Problem Solved By users',
          count: { $sum: '$problemSolved' }
        }
      }
    ]);

//5.Find all the mentors with who has the mentee's count more than 15

db.getCollection('mentors').aggregate(
    [{ $match: { menteeCount: { $gt: 15 } } }]);

//6.Find the number of users who are absent and task is not submitted between 15 oct-2023 and 31-oct-2023

db.getCollection('Attendence').aggregate(
    [
      {
        $lookup: {
          from: 'topics',
          localField: 'topicId',
          foreignField: 'topicId',
          as: 'Absent'
        }
      },
      {
        $lookup: {
          from: 'tasks',
          localField: 'userId',
          foreignField: 'userId',
          as: 'Task-notSubmitted'
        }
      },
      { $unwind: '$Task-notSubmitted' },
      { $unwind: '$Absent' },
      {
        $match: {
          attended: false,
          'Absent.topicDate': {
            $gte: '2023-10-15',
            $lte: '2023-10-31'
          }
        }
      },
      {
        $match: {
          'Task-notSubmitted.dueDate': {
            $gte: '2023-10-15',
            $lte: '2023-10-31'
          },
          'Task-notSubmitted.submitted': false
        }
      },
      { $project: { _id: 0, 'Absent._id': 0 } },
      { $project: { 'Task-notSubmitted._id': 0 } }
    ]);
