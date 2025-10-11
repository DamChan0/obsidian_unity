## 구현

- room 클래스를 상속 받아서 구현 할 때는 반드시 abstract로 구현해야한다 
- 이때 RoobDatabase()를 상속 받고 그 안의 함수들도 abstract로 구현해야함
	- DAO 메서드: imageDao() 메서드도 abstract로 선언하는 이유는 Room이 이 메서드의 구현체를 자동으로 생성하기 때문이다.
	- e.g
	```kotlin
	package com.example.grid.data.local
	
	import android.media.Image
	import android.content.Context
	import androidx.room.Database
	import androidx.room.Room
	import androidx.room.RoomDatabase
	
	
	@Database(entities = [ImageEntity::class], version = 1)
	abstract class AppDatabase : RoomDatabase() {
	
	    abstract fun imageDao(): ImageDao
	
	    companion object {
	        @Volatile
	        private var INSTANCE: AppDatabase? = null
	
	        fun getDatabase(context: Context): AppDatabase {
	            return INSTANCE ?: synchronized(this) {
	                val instance = Room.databaseBuilder(
	                    context.applicationContext,
	                    AppDatabase::class.java,
	                    "image_database" // DB 파일명
	                ).build()
	                INSTANCE = instance
	                instance
	            }
	        }
	    }
	}
	```
	
