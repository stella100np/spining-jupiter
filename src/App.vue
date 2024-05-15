<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// 引入CSS2渲染器CSS2DRenderer和CSS2模型对象CSS2DObject
import {
	CSS2DRenderer,
	CSS2DObject,
} from "three/addons/renderers/CSS2DRenderer.js";

import ballImg from "./assets/ball.png";
const width = window.innerWidth; //宽度
const height = window.innerHeight; //高度

const canvasContainer = ref(null);
const infoContainer = ref(null);
let scene, camera, renderer, labelRenderer, controls, spriteGroup;
let isRotating = true;

//第四步调用渲染函数进行渲染
function animate(time) {
	requestAnimationFrame(animate);
	controls.update();
	isRotating && spriteGroup.rotateZ(0.01);
	renderer.render(scene, camera);
}

onMounted(() => {
	//第一步创建场景、相机、渲染器

	scene = new THREE.Scene();
	camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
	renderer = new THREE.WebGLRenderer({ antialias: true });
	renderer.setPixelRatio(window.devicePixelRatio);
	renderer.setSize(width, height); //设置three.js渲染区域的尺寸(像素px)
	canvasContainer.value.appendChild(renderer.domElement);

	// 初始化CSS2DRenderer
	labelRenderer = new CSS2DRenderer();
	labelRenderer.setSize(width, height);
	labelRenderer.domElement.style.position = "absolute";
	labelRenderer.domElement.style.top = "0px";
	labelRenderer.domElement.style["pointer-events"] = "none";
	infoContainer.value.appendChild(labelRenderer.domElement);

	// 将球体和圆圈添加到场景中
	const sphere = getSphere();
	scene.add(sphere);
	const circle = getCircle();
	scene.add(circle);
	spriteGroup = getSpriteGroup();
	scene.add(spriteGroup);

	scene.add(new THREE.AxesHelper(150)); // 添加一个xyz坐标轴展示器

	controls = new OrbitControls(camera, renderer.domElement);
	camera.position.z = 5;
	controls.update();
	animate(0);
});

function getSphere() {
	const geometry = new THREE.SphereGeometry(1, 32, 32);
	const material = new THREE.MeshBasicMaterial({ color: 0xffe4b5 });
	const sphere = new THREE.Mesh(geometry, material);
	return sphere;
}

function getCircle() {
	const arcCurve = new THREE.ArcCurve(0, 0, 4, 0, 2 * Math.PI);
	const geometry = new THREE.BufferGeometry().setFromPoints(
		arcCurve.getPoints(50),
	);
	const material = new THREE.LineBasicMaterial({ color: 0x00c5cd });
	const line = new THREE.Line(geometry, material);
	return line;
}

function getSpriteGroup() {
	const textureLoader = new THREE.TextureLoader();
	const colorTexture = textureLoader.load(ballImg); // 如果需要的话
	const spriteMaterial = new THREE.SpriteMaterial({
		map: colorTexture,
		transparent: true,
	});
	const spriteGroup = new THREE.Group();
	const radius = 4; // 控制圆的半径
	for (let i = 0; i < 10; i++) {
		const angle = (i / 10) * Math.PI * 2; // 计算每个精灵的角度位置
		const sprite = new THREE.Sprite(spriteMaterial.clone());
		sprite.scale.set(0.5, 0.5, 0.5); // 设置精灵大小
		sprite.position.set(radius * Math.cos(angle), radius * Math.sin(angle), 0); // 根据角度设置位置
		spriteGroup.add(sprite);
	}
	return spriteGroup;
}

const showInfoBox = (object) => {
	const infoDiv = document.createElement("div");
	infoDiv.className = "info-box"; // 自定义样式
	infoDiv.textContent = `点击的精灵信息：`;

	const label = new CSS2DObject(infoDiv);
	label.position.copy(object.getWorldPosition(new THREE.Vector3()));
	object.add(label);

	// 由于CSS2DRenderer不参与动画循环，所以只需一次渲染
	labelRenderer.render(scene, camera);
};

const onCanvasClick = (event) => {
	// .offsetY、.offsetX以canvas画布左上角为坐标原点,单位px
	const px = event.offsetX;
	const py = event.offsetY;
	//屏幕坐标px、py转WebGL标准设备坐标x、y
	//width、height表示canvas画布宽高度

	const x = (px / width) * 2 - 1;
	const y = -(py / height) * 2 + 1;
	//创建一个射线投射器`Raycaster`
	const raycaster = new THREE.Raycaster();
	//.setFromCamera()计算射线投射器`Raycaster`的射线属性.ray
	// 形象点说就是在点击位置创建一条射线，射线穿过的模型代表选中
	raycaster.setFromCamera(new THREE.Vector2(x, y), camera);
	//.intersectObjects([mesh1, mesh2, mesh3])对参数中的网格模型对象进行射线交叉计算
	// 未选中对象返回空数组[],选中一个对象，数组1个元素，选中两个对象，数组两个元素
	const intersects = raycaster.intersectObjects(spriteGroup.children);
	console.log("射线器返回的对象", intersects);
	// intersects.length大于0说明，说明选中了模型
	if (intersects.length > 0) {
		// 选中模型的第一个模型，设置为红色
		isRotating = false;
		showInfoBox(intersects[0].object); // 显示信息框
		// intersects[0].object.material.color.set(0xff0000);
	} else {
		isRotating = true;
		console.log("xxxxxx");
	}
};
</script>

<template>
	<div ref="canvasContainer" @click="onCanvasClick"></div>
	<div ref="infoContainer" class="info-box-container">
	</div>
</template>

<style scoped></style>
