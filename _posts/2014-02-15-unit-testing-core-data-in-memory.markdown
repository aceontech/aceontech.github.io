---
layout: post
title:  "How to set up in-memory unit testing for Core Data"
date:   2014-02-15 11:25:00
categories: objc ios coredata
image: ios7-logo.png
---

So, you're using Core Data in your iOS app. But what about unit testing (you are testing, right)? Setting up Core Data
in-memory is easy:

<script src="https://gist.github.com/aceontech/8860058.js"></script>

For easy re-use, shove it into a category:

```objc
@implementation NSManagedObjectContext (Testing)

+ (instancetype)testing_inMemoryContext:(NSManagedObjectContextConcurrencyType)concurrencyType error:(NSError **)error
{
    // ObjectModel from any models in app bundle
    NSManagedObjectModel *managedObjectModel = [NSManagedObjectModel mergedModelFromBundles:nil];

    // Coordinator with in-mem store type
    NSPersistentStoreCoordinator *coordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:managedObjectModel];
    [coordinator addPersistentStoreWithType:NSInMemoryStoreType configuration:nil URL:nil options:nil error:error];

    // Context with private queue (for example)
    NSManagedObjectContext *context = [[NSManagedObjectContext alloc] initWithConcurrencyType:concurrencyType];
    context.persistentStoreCoordinator = coordinator;

    return context;
}

@end
```