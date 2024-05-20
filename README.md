# Getting-and-Cleaning-Data-Course-Project
Getting and Cleaning Data Course Project

This is a project for peer review
Steps:

1. Load packages that used to deal with data
    packages <- c("data.table", "reshape2")
    sapply(packages, require, character.only=TRUE, quietly=TRUE)
2. We will set the directory for our work and then save the path in new variable
    path <- getwd()
3. Data from the url we set it in new variable and then download it to our path and unzip them
    url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(url, file.path(path, "dataFiles.zip"))
    unzip(zipfile = "dataFiles.zip")
4. Load activity labels + features
    activityLabels <- fread(file.path(path, "UCI HAR Dataset/activity_labels.txt"), col.names = c("classLabels", "activityName"))
    features <- fread(file.path(path, "UCI HAR Dataset/features.txt"), col.names = c("index", "featureNames"))
    featuresWanted <- grep("(mean|std)\\(\\)", features[, featureNames])
    measurements <- features[featuresWanted, featureNames]
    measurements <- gsub('[()]', '', measurements)
5. Load train datasets
    train <- fread(file.path(path, "UCI HAR Dataset/train/X_train.txt"))[, featuresWanted, with = FALSE]
    data.table::setnames(train, colnames(train), measurements)
    trainActivities <- fread(file.path(path, "UCI HAR Dataset/train/Y_train.txt"), col.names = c("Activity"))
    trainSubjects <- fread(file.path(path, "UCI HAR Dataset/train/subject_train.txt"), col.names = c("SubjectNum"))
    train <- cbind(trainSubjects, trainActivities, train)
6. Load test datasets
    test <- fread(file.path(path, "UCI HAR Dataset/test/X_test.txt"))[, featuresWanted, with = FALSE]
    data.table::setnames(test, colnames(test), measurements)
    testActivities <- fread(file.path(path, "UCI HAR Dataset/test/Y_test.txt"), col.names = c("Activity"))
    testSubjects <- fread(file.path(path, "UCI HAR Dataset/test/subject_test.txt"), col.names = c("SubjectNum"))
    test <- cbind(testSubjects, testActivities, test)
7. merge datasets
    combined <- rbind(train, test)
8. Convert classLabels to activityName basically. More explicit
    combined[["Activity"]] <- factor(combined[, Activity], levels = activityLabels[["classLabels"]], labels = activityLabels[["activityName"]])
    combined[["SubjectNum"]] <- as.factor(combined[, SubjectNum])
    combined <- reshape2::melt(data = combined, id = c("SubjectNum", "Activity"))
    combined <- reshape2::dcast(data = combined, SubjectNum + Activity ~ variable, fun.aggregate = mean)
9. Generate the new file
    data.table::fwrite(x = combined, file = "tidyData.txt", quote = FALSE)

You can see the R script in the same repository. You can do that with some other libraries. Have fun!!