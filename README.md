plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
}

android {
    compileSdk 34

    defaultConfig {
        applicationId "com.exemplo.estoque"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    buildFeatures {
        compose true
    }

    composeOptions {
        kotlinCompilerExtensionVersion '1.4.3'
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation "androidx.core:core-ktx:1.12.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.6.2"
    implementation "androidx.activity:activity-compose:1.8.2"
    implementation "androidx.compose.ui:ui:1.5.4"
    implementation "androidx.compose.material3:material3:1.1.2"

    // Room
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
    implementation "androidx.room:room-ktx:2.6.1"

    // Lifecycle
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2"

    // Compose Navigation
    implementation "androidx.navigation:navigation-compose:2.7.7"
}

@Entity(tableName = "itens")
data class ItemEstoque(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val nome: String,
    val quantidade: Int,
    val descricao: String
)

@Entity(tableName = "itens")
data class ItemEstoque(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val nome: String,
    val quantidade: Int,
    val descricao: String
)

class ItemRepository(private val dao: ItemDao) {
    val itens: Flow<List<ItemEstoque>> = dao.getAll()

    suspend fun add(item: ItemEstoque) = dao.insert(item)
    suspend fun update(item: ItemEstoque) = dao.update(item)
    suspend fun delete(item: ItemEstoque) = dao.delete(item)
}

class ItemViewModel(application: Application) : AndroidViewModel(application) {
    private val db = Room.databaseBuilder(application, AppDatabase::class.java, "estoque.db").build()
    private val repo = ItemRepository(db.itemDao())

    val itens = repo.itens.asLiveData()

    fun addItem(item: ItemEstoque) = viewModelScope.launch {
        repo.add(item)
    }

    fun updateItem(item: ItemEstoque) = viewModelScope.launch {
        repo.update(item)
    }

    fun deleteItem(item: ItemEstoque) = viewModelScope.launch {
        repo.delete(item)
    }
}

@Composable
fun TelaListaItens(viewModel: ItemViewModel) {
    val itens by viewModel.itens.observeAsState(emptyList())
    val context = LocalContext.current

    LazyColumn {
        items(itens) { item ->
            Card(
                modifier = Modifier
                    .padding(8.dp)
                    .fillMaxWidth(),
                elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
            ) {
                Column(Modifier.padding(16.dp)) {
                    Text("Item: ${item.nome}", style = MaterialTheme.typography.titleLarge)
                    Text("Quantidade: ${item.quantidade}")
                    Text("Descrição: ${item.descricao}")

                    Row {
                        Button(onClick = { viewModel.deleteItem(item) }) {
                            Text("Excluir")
                        }
                        Spacer(modifier = Modifier.width(8.dp))
                        // Adicionar edição aqui, se necessário
                    }
                }
            }
        }
    }
}
