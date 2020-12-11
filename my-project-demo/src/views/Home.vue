<template>
  <div class="home">
    <div style="width:500px;height:500px;" ref="previewContainer" id="previewContainer"></div>
    <div>
      <input placeholder="X" v-model="addX" style="height:30px;line-height:30px;margin-right:20px;margin-left:20px;text-indent:10px;"/>
      <input placeholder="Y" v-model="addY" style="height:30px;line-height:30px;margin-right:20px;text-indent:10px;"/>
    </div>
    <div style="width:0;height:0;overflow:hidden;">
      <ul class="preview-coordinate-ul" id="preview-coordinate-ul" :style="`width:${roomRow*50}px;height:${roomColumn*50}px`">
        <li v-for="y in roomColumn" :key="'y' + y">
          <span v-for="x in roomRow" :key="'x' + x" class="preview-coordinate">X<em>{{x}}</em>Y<em>{{y}}</em></span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import html2canvas from "html2canvas";
export default {
  name: 'Home',
  data() {
    return {
      roomRow: 4,
      roomColumn: 7,
      cabinetList: [{
        id: 1,
        x: 2,
        y: 3
      }, {
        id: 2,
        x: 4,
        y: 7
      }, {
        id: 3,
        x: 4,
        y: 5
      }],
      addX: "",
      addY: "",
      renderer: null,
      scene: null,
      camera: null,
      controls: null,
      currentMesh: null
    }
  },
  watch: {
    addX: function(newVal, oldVal) {
      if (newVal && this.addY) {
        this.isRepeatAndPreview(); 
      }
    },
    addY: function(newVal, oldVal) {
      if (newVal && this.addX) {
        this.isRepeatAndPreview(); 
      }
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.init3D();
      this.animate();
    })
  },
  methods: {
    init3D() {
      this.renderer = new THREE.WebGLRenderer(); //渲染器
      let container = document.getElementById('previewContainer');
      this.renderer.setSize(container.offsetWidth, container.offsetHeight);
      container.appendChild(this.renderer.domElement);

      this.scene = new THREE.Scene(); //场景
      this.scene.background = new THREE.Color(0xffffff);

      this.camera = new THREE.PerspectiveCamera(90, container.offsetWidth/container.offsetHeight, 1, 20000); //相机
      this.controls = new OrbitControls(this.camera, this.renderer.domElement); //控制器
      this.controls.enableRotate = false;

      // 放大100倍，避免线宽问题
      let wid = this.roomRow * 100;
      let hei = this.roomColumn * 100;
      this.camera.position.x = wid/2;
      this.camera.position.y = wid > hei ? Math.ceil(wid/2) + 100 : Math.ceil(hei/2) + 100;  //长边的一半，保证看到全部
      this.camera.position.z = hei/2;
      this.camera.lookAt(new THREE.Vector3(wid/2, 0, hei/2));
      this.controls.target = new THREE.Vector3(wid/2, 0, hei/2);
      this.scene.add(this.camera);

      this.createCoordinate(wid, hei); //创建底部坐标
      this.drawX(wid, hei);
      this.drawY(wid, hei);
      this.drawInitCabinet();

      this.renderer.render(this.scene, this.camera);
    },
    async createCoordinate(wid, hei) {
      await html2canvas(document.getElementById('preview-coordinate-ul')).then(  //截图
        canvas => {
          document.getElementsByClassName('home')[0].appendChild(canvas);
          let texture = new THREE.CanvasTexture(canvas); //纹理贴图
          let geometry = new THREE.PlaneGeometry(wid, hei); //平面体
          let material = new THREE.MeshBasicMaterial({ //基础网格材质
            map: texture
          })
          let mesh = new THREE.Mesh(geometry, material); //网格物体,默认平行与 x y 平面
          mesh.rotation.x = - Math.PI / 2 ; //旋转到 x z 平面
          mesh.position.set(wid/2, -1, hei/2);
          this.scene.add(mesh);
        }
      )
    },
    drawX(wid, hei) {
      let geometry = new THREE.Geometry();
      geometry.vertices.push(new THREE.Vector3(0, 0, 0));
      geometry.vertices.push(new THREE.Vector3(wid, 0, 0));
      for (let i=0; i<=hei/100; i++) {
        let color = 0xbbbbbb;
        let line = new THREE.Line(
          geometry,
          new THREE.LineBasicMaterial({color: color, opacity: 1})
        );
        line.position.z = i * 100;
        this.scene.add(line);
      }
    },
    drawY(wid, hei) {
      let geometry = new THREE.Geometry();
      geometry.vertices.push(new THREE.Vector3(0, 0, 0));
      geometry.vertices.push(new THREE.Vector3(0, 0, hei));
      for (let i=0; i<=wid/100; i++) {
        let color = 0xbbbbbb;
        let line = new THREE.Line(
          geometry,
          new THREE.LineBasicMaterial({color: color, opacity: 1})
        );
        line.position.x = i * 100;
        this.scene.add(line);
      }
    },
    drawInitCabinet() {
      for (let i=0; i<this.cabinetList.length; i++) {
        let geometry = new THREE.PlaneGeometry(80, 80); 
        let material = new THREE.MeshBasicMaterial({ 
          color: 0x000000
        });
        let mesh = new THREE.Mesh(geometry, material);
        mesh.rotation.x = - Math.PI / 2 ;
        mesh.position.set((this.cabinetList[i].x-1)*100 + 50, 0, (this.cabinetList[i].y-1)*100 + 50);
        this.scene.add(mesh);
      }
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
    isRepeatAndPreview() {
      let findFlag = 0;
      for (let i=0; i<this.cabinetList.length; i++) {
        if (this.addX == this.cabinetList[i].x && this.addY == this.cabinetList[i].y) {
          findFlag = 1;
          break;
        }
      }
      if (findFlag == 1) {
        this.scene.remove(this.currentMesh);
        this.$message({
          message: '对应位置已使用',
          type: 'warning'
        });
      } else {
        if (this.addX > 0 && this.addX <= this.roomRow && this.addY > 0 && this.addY <= this.roomColumn) {
          this.scene.remove(this.currentMesh);
          let geometry = new THREE.PlaneGeometry(80, 80); 
          let material = new THREE.MeshBasicMaterial({ 
            color: 0x0087ed
          });
          this.currentMesh = new THREE.Mesh(geometry, material);
          this.currentMesh.rotation.x = - Math.PI / 2 ;
          this.currentMesh.position.set((this.addX-1)*100 + 50, 0, (this.addY-1)*100 + 50);
          this.scene.add(this.currentMesh);
        } else {
          this.scene.remove(this.currentMesh);
          if (this.addX <= 0 || this.addX > this.roomRow) {
            this.$message({
              message: '请输入正确行位置',
              type: 'warning'
            });
          } else if (this.addY <= 0 || this.addY > this.roomColumn) {
            this.$message({
              message: '请输入正确列位置',
              type: 'warning'
            });
          } else {
            this.$message({
              message: '请输入正确位置',
              type: 'warning'
            });
          }
        }
      }
    }
  }
}
</script>
<style>
.preview-coordinate-ul {
  white-space: nowrap;
}

.preview-coordinate {
  display: inline-block;
  width: 50px;
  text-align: center;
  height: 50px;
  line-height: 50px;
  font-size: 14px;
}

.preview-coordinate em {
  color: red;;
}
</style>
