# Getting-and-Cleaning-Data

library(plyr)
library(dplyr)
library(data.table)

fichiertemporaire<- tempfile()

download.file("http://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",temp)
unzip(fichiertemporaire, list = TRUE) #This provides the list of variables and I choose the ones that are applicable for this data set
yTest <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/test/y_test.txt"))
XTest <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/test/X_test.txt"))
SubjectTest <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/test/subject_test.txt"))
yTrain <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/train/y_train.txt"))
XTrain <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/train/X_train.txt"))
SubjectTrain <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/train/subject_train.txt"))
fichier <- read.table(unzip(fichiertemporaire, "UCI HAR Dataset/features.txt"))
unlink(fichiertemporaire) 


colnames(XTrain) <- t(fichier[2])
colnames(XTest) <- t(fichier[2])

XTrain$activities <- yTrain[, 1]
XTrain$participants <- SubjectTrain[, 1]
XTest$activities <- yTest[, 1]
XTest$participants <- SubjectTest[, 1]

#Part 1
Master <- rbind(XTrain, XTest)
duplicated(colnames(Master))
Master <- Master[, !duplicated(colnames(Master))]

#Part 2
Mean <- grep("mean()", names(Master), value = FALSE, fixed = TRUE)
Mean <- append(Mean, 471:477)
InstrumentMeanMatrix <- Master[Mean]
STD <- grep("std()", names(Master), value = FALSE)
InstrumentSTDMatrix <- Master[STD]

#Part 3
Master$activities <- as.character(Master$activities)
Master$activities[Master$activities == 1] <- "Walking"
Master$activities[Master$activities == 2] <- "Walking Upstairs"
Master$activities[Master$activities == 3] <- "Walking Downstairs"
Master$activities[Master$activities == 4] <- "Sitting"
Master$activities[Master$activities == 5] <- "Standing"
Master$activities[Master$activities == 6] <- "Laying"
Master$activities <- as.factor(Master$activities)

#Part 4

#Part 5
Master.dt <- data.table(Master)
TidyData <- Master.dt[, lapply(.SD, mean), by = 'participants,activities']
write.table(TidyData, file = "Tidy.txt", row.names = FALSE)
