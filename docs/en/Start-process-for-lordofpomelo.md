Lordofpomelo is developed under Pomelo framework, the boot process use Pomelo's startup mode,  please read the reference before reading this wiki [pomelo start link].

App.js is the entrance of lordofpomelo, it is resposible for the configuration of current thread. The start process of lordofpomelo mainly includes two different parts: start the master server, start other servers.

###  Loading Components
In lordofpemelo, we use multiple components, these components loaded at start time, and provide extended functions, like: data statistics, routing, game initialization and so on.

In lordofpomelo we use script-based statistics method, by running custom scripts, the monitor module can collect servers' running data and information to generate reports.

``` javascript
	app.registerAdmin(require('./app/modules/sceneInfo'), {app:app});
	app.registerAdmin(require('./app/modules/onlineUser'), {app:app});
```

Lordofpomelo startup also load area configuration to create the mapping between scenes and area servers.

``` javascript
	if (app.serverType !== 'master') {
		var areas = app.get('servers').area;
		var areaIdMap = {};
		for(var id in areas){
			areaIdMap[areas[id].area] = areas[id].id;
		}
		app.set('areaIdMap', areaIdMap);
	}
```

In order to routing among the servers, lordofpomelo loaded custom routing components.It uses scene mapping to make sure players can be routed to correct area server.

``` javascript
	app.route('area', routeUtil.area);
	app.route('connector', routeUtil.connector);
```

In addition to the server generic configuration, app is also responsible for the initialization of the different services, such as global server initialization and the area server initialization, as well as path finding servers. The initialization process will be difference among different servers, depending on the type of the server.

``` javascript
app.configure('production|development', 'area', function(){
	var areaId = app.get('curServer').area;
	if(!areaId || areaId < 0) {
		throw new Error('load area config failed');
	}
	world.init(dataApi.area.all());
	area.init(dataApi.area.findById(areaId));
});
```

Path finding services and mysql initialization.

``` javascript
app.configure('production|development', 'path', function(){
	pathRemote.init();
});

app.configure('production|development', 'area|auth|connector|master', function() {
	var dbclient = require('./app/dao/mysql/mysql').init(app);
	app.load(pomelo.sync, {path:__dirname + '/app/dao/mapping', dbclient: dbclient});
});
```

### Server startup

Master component is responsible for starting all other servers. The startup process is divided into two steps. First, master will start all other server via ssh. After a server was started, the monitor component will connect to master, indicating that the server is booted. Second, after all the servers are booted, master will call all the servers to invoke the afterStart interface, to do after start jobs.