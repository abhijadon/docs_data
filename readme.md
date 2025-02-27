# DATABASE  to fetch all application records 
let cursor = db.applications.find(); // Fetch all application records

cursor.forEach(app => {
    let institute = db.institutes.findOne({ name: app.institute_name }); // Find matching institute

    if (institute) {
        db.applications.updateOne(
            { _id: app._id }, // Match application by its own _id
            { $set: { institute_name: institute._id } } // Replace institute_name with ObjectId
        );
        print(Updated application ${app._id}: Replaced '${app.institute_name}' with ${institute._id});
    } else {
        print(No matching institute found for ${app.institute_name});
    }
});

### update
db.applications.updateMany(
    {}, // Update all documents
    { $set: { removed: false, enabled: true } } // Set required fields
);

### rename
db.applications.updateMany(
    {}, // Update all documents
    { $rename: { "enter_specialization": "subcourse" } } // Rename field without changing data
);

### move
mv /Users/apple/Desktop/~/Users/apple/Desktop/mongo_backup/new_erp ~/Desktop/

### restore
mongorestore --uri="mongodb+srv://abhishek:abhishek2024@cluster0.dq77zco.mongodb.net/erp_new?retryWrites=true&w=majority" --dir=~/Desktop/new_erp/

### consider only two data from the object and remove some field
db.applications.find().forEach((doc) => { 
    if (doc.customfields && typeof doc.customfields === "object") { 
        let customData = doc.customfields; // customfields object ko store karein

        db.applications.updateOne(
            { _id: doc._id }, // Match the document by _id
            { 
                $set: customData,  // customfields ke andar ka data merge karein
                $unset: { customfields: "" } // customfields field remove karein
            }
        );

        print(Updated document ${doc._id}: Merged customfields and removed the field.);
    }
});
### bulk data update
let bulkOps = []; // Initialize bulk operations array
let batchSize = 500; // Batch size for efficient processing
let count = 0;

db.applications.find().forEach(app => {
    let university = db.universities.findOne({ name: app.university_name });

    if (university) {
        bulkOps.push({
            updateOne: {
                filter: { _id: app._id },
                update: { $set: { university_name: university._id } }
            }
        });

        count++;

        // Execute batch update when batch size is reached
        if (bulkOps.length >= batchSize) {
            db.applications.bulkWrite(bulkOps);
            print(Updated ${count} applications in bulk);
            bulkOps = []; // Reset batch
        }
    }
});
### update many
db.subcourses.find().forEach(subcourseRecord => {
    let result = db.applications.updateMany(
        { subcourse: subcourseRecord.name }, // Find applications where subcourse matches
        { $set: { subcourse: subcourseRecord._id } } // Replace subcourse with ObjectId
    );

    print(Updated ${result.modifiedCount} applications for subcourse '${subcourseRecord.name}');
});

// Execute remaining updates
if (bulkOps.length > 0) {
    db.applications.bulkWrite(bulkOps);
    print(Updated final ${bulkOps.length} applications);
}

### to modify the date 
let leadIds = [
    "718462", "243600", "560348", "248807", "170460", "732404", "433234", "964833",
    "287250", "476375", "317778", "189572", "267074", "218227", "530953", "918782",
    "837006", "712117", "153933", "519638", "411806", "918497", "864916", "661832",
    "516072", "239256", "620619", "577313", "170603", "491440", "914246", "691499",
    "963546", "788996", "930933", "888185", "758528", "162366", "785786", "977426",
    "958714", "782489", "332877", "825980", "393799", "941937", "246988", "559936",
    "940912", "452245", "544432", "109943", "437856", "511905", "236355", "347791",
    "167190", "922368", "685639", "242299", "354095", "866136", "176502", "235029",
    "707082", "169692", "124926", "699783", "309235", "783756", "227043", "269389",
    "857191", "161398", "792184", "665183", "349611", "398336", "260959", "769695",
    "170563", "937297", "938993", "547545", "671956", "840109", "140201", "967934",
    "981655", "221815", "814190", "604709", "689734", "895351", "520970", "386628",
    "481823", "100751", "641370", "975206", "699379", "589141", "116592", "483175",
    "810660", "345815", "869378", "490598", "403594", "716981", "692554", "735631",
    "703061", "560308", "113491", "262358", "603625", "416493", "677214", "158662",
    "509691", "366033", "329911", "808059", "957802", "740846", "861815", "775066",
    "523803", "321826", "289281", "109996", "312790", "384224", "433885", "597667",
    "777052", "263598", "327998", "596935", "878698", "757821", "888873", "872304",
    "395110", "329918", "551315", "457150", "963344", "807869", "971843", "355835",
    "724255", "383907", "258565", "285614"
];

// Set createdAt date (Modify this if required)
let newCreatedAt = new ISODate("2023-07-01T00:00:00Z"); // Example date: 1st July 2023

db.applications.updateMany(
    { lead_id: { $in: leadIds } }, // Match all given lead_id values
    { $set: { createdAt: newCreatedAt } } // Update createdAt
);
### to do it in array
let cursor = db.applications.find(); // Fetch all application records

cursor.forEach(app => {
    let updateFields = {};

    // Find matching subcourse from subcourses collection
    let subcourseRecord = db.subcourses.findOne({ name: app.subcourse });
    if (subcourseRecord) {
        updateFields.subcourse = subcourseRecord._id;
    }

    // Ensure previousData exists and is an array
    if (app.previousData && Array.isArray(app.previousData) && app.previousData.length > 0) {
        let updatedPreviousData = app.previousData.map((entry) => {
            let updatedEntry = { ...entry };

            // Find matching installment_type from InstallmentType collection
            if (entry.installment_type) {
                let installmentRecord = db.installmenttypes.findOne({ name: entry.installment_type });
                if (installmentRecord) {
                    updatedEntry.installment_type = installmentRecord._id;
                }
            }

            // Find matching paymentStatus from PaymentType collection
            if (entry.paymentStatus) {
                let paymentStatusRecord = db.paymenttypes.findOne({ name: entry.paymentStatus });
                if (paymentStatusRecord) {
                    updatedEntry.paymentStatus = paymentStatusRecord._id;
                }
            }

            // Find matching payment_mode from PaymentMode collection
            if (entry.payment_mode) {
                let paymentModeRecord = db.paymentmodes.findOne({ name: entry.payment_mode });
                if (paymentModeRecord) {
                    updatedEntry.payment_mode = paymentModeRecord._id;
                }
            }

            // Find matching payment_type from PaymentStatus collection
            if (entry.payment_type) {
                let paymentTypeRecord = db.paymentstatuses.findOne({ name: entry.payment_type });
                if (paymentTypeRecord) {
                    updatedEntry.payment_type = paymentTypeRecord._id;
                }
            }

            return updatedEntry;
        });

        updateFields.previousData = updatedPreviousData;
    } else {
        print(Skipping application ${app._id} - previousData is missing or not an array);
    }

    // Update the application record if there are fields to update
    if (Object.keys(updateFields).length > 0) {
        db.applications.updateOne(
            { _id: app._id }, // Match application by its own _id
            { $set: updateFields } // Update all matching fields
        );

        print(Updated application ${app._id}: ${JSON.stringify(updateFields)});
    } else {
        print(No matching records found for application ${app._id});
    }
});
### find the values for greater than or less than or equal to $eq $ne $gt $gte $lt $lte $in $nin
db.payments.countDocuments({paid_amount:{$gte:5000}})

### in operator
db.payments.countDocuments({paid_amount:{$in:[5000, 6000,3000]}})

### limit()
db.payments.find({ paid_amount: { $lt: 5000 } }).limit(5).itcount()

db.payments.find({ paid_amount: { $lt: 5000 } }).limit(5).itcount()
db.payments.find({paid_amount: {$lt: 5000}}).limit(2).skip(1)

### sorting data(-1 for descending and 1 for ascending )
db.payments.find({paid_amount:{$lt:5000}}).sort({paid_amount:-1}).limit(5)

### $and $or $nor operator
 db.payments.find({$nor:[{paid_amount:{$lt:5000}},{'institute_name':'DES'}]})

### db.payments.find({paid_amount:{$exists:true}}).count()

### rename a field
 erp_sode> db.payments.updateOne(
... {student_name: 'Atanu Adhikary'},
... {$set:{name:'Khushboo Das'}}
... )

### to update in bulk fastly 
let cursor = db.applications.find(); // Fetch all application records

cursor.forEach(app => {
    let updateFields = {};

    // Find matching subcourse from subcourses collection
    let subcourseRecord = db.subcourses.findOne({ name: app.subcourse });
    if (subcourseRecord) {
        updateFields.subcourse = subcourseRecord._id;
    }

    // Ensure previousData exists and is an array
    if (app.previousData && Array.isArray(app.previousData) && app.previousData.length > 0) {
        let updatedPreviousData = app.previousData.map((entry) => {
            let updatedEntry = { ...entry };

            // Find matching installment_type from InstallmentType collection
            if (entry.installment_type) {
                let installmentRecord = db.installmenttypes.findOne({ name: entry.installment_type });
                if (installmentRecord) {
                    updatedEntry.installment_type = installmentRecord._id;
                }
            }

            // Find matching paymentStatus from PaymentType collection
            if (entry.paymentStatus) {
                let paymentStatusRecord = db.paymenttypes.findOne({ name: entry.paymentStatus });
                if (paymentStatusRecord) {
                    updatedEntry.paymentStatus = paymentStatusRecord._id;
                }
            }

            // Find matching payment_mode from PaymentMode collection
            if (entry.payment_mode) {
                let paymentModeRecord = db.paymentmodes.findOne({ name: entry.payment_mode });
                if (paymentModeRecord) {
                    updatedEntry.payment_mode = paymentModeRecord._id;
                }
            }

            // Find matching payment_type from PaymentStatus collection
            if (entry.payment_type) {
                let paymentTypeRecord = db.paymentstatuses.findOne({ name: entry.payment_type });
                if (paymentTypeRecord) {
                    updatedEntry.payment_type = paymentTypeRecord._id;
                }
            }

            return updatedEntry;
        });

        updateFields.previousData = updatedPreviousData;
    } else {
        print(Skipping application ${app._id} - previousData is missing or not an array);
    }

    // Update the application record if there are fields to update
    if (Object.keys(updateFields).length > 0) {
        db.applications.updateOne(
            { _id: app._id }, // Match application by its own _id
            { $set: updateFields } // Update all matching fields
        );

        print(Updated application ${app._id}: ${JSON.stringify(updateFields)});
    } else {
        print(No matching records found for application ${app._id});
    }
});

Abhishek Jadon, Thu 23:57
https://meet.google.com/wxd-fetd-sxx?authuser=0

Abhishek Jadon, Sat 11:06
mongodb+srv://abhishek:abhishek2024@cluster0.dq77zco.mongodb.net/erp_sode?retryWrites=true&w=majority

Abhishek Jadon, 1 min

// Fetch all applications
let cursor = db.applications.find().toArray();

// Fetch all required mapping data in bulk for efficiency
let subcourseMap = db.subcourses.find().toArray().reduce((acc, subcourse) => {
    acc[subcourse.name] = subcourse._id;
    return acc;
}, {});

let installmentMap = db.installmenttypes.find().toArray().reduce((acc, installment) => {
    acc[installment.name] = installment._id;
    return acc;
}, {});

let paymentStatusMap = db.paymenttypes.find().toArray().reduce((acc, paymentType) => {
    acc[paymentType.name] = paymentType._id;
    return acc;
}, {});

let paymentModeMap = db.paymentmodes.find().toArray().reduce((acc, paymentMode) => {
    acc[paymentMode.name] = paymentMode._id;
    return acc;
}, {});

let paymentTypeMap = db.paymentstatuses.find().toArray().reduce((acc, paymentStatus) => {
    acc[paymentStatus.name] = paymentStatus._id;
    return acc;
}, {});

// Process applications
cursor.forEach(app => {
    let updateFields = {}; // Object to store fields to be updated

    // Update subcourse field using preloaded map
    if (app.subcourse && subcourseMap[app.subcourse]) {
        updateFields.subcourse = subcourseMap[app.subcourse];
    }

    // Ensure previousData exists and is an array before processing
    if (app.previousData && Array.isArray(app.previousData) && app.previousData.length > 0) {
        let updatedPreviousData = app.previousData.map((entry) => {
            let updatedEntry = { ...entry };

            // Update installment_type
            if (entry.installment_type && installmentMap[entry.installment_type]) {
                updatedEntry.installment_type = installmentMap[entry.installment_type];
            }

            // Update paymentStatus
            if (entry.paymentStatus && paymentStatusMap[entry.paymentStatus]) {
                updatedEntry.paymentStatus = paymentStatusMap[entry.paymentStatus];
            }

            // Update payment_mode
            if (entry.payment_mode && paymentModeMap[entry.payment_mode]) {
                updatedEntry.payment_mode = paymentModeMap[entry.payment_mode];
            }

            // Update payment_type
            if (entry.payment_type && paymentTypeMap[entry.payment_type]) {
                updatedEntry.payment_type = paymentTypeMap[entry.payment_type];
            }

            return updatedEntry;
        });

        updateFields.previousData = updatedPreviousData;
    } else {
        print(Skipping application ${app._id} - previousData is missing or not an array);
    }

    // Update multiple applications where needed
    if (Object.keys(updateFields).length > 0) {
        let result = db.applications.updateMany(
            { _id: app._id }, // Match application by _id
            { $set: updateFields } // Apply updates
        );

        print(Updated ${result.modifiedCount} applications for ID: ${app._id});
    } else {
        print(No updates needed for application ${app._id});
    }
});




